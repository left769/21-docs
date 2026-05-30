# Резервування провайдерів (Dual ISP Failover) - Шпаргалка
 
> Курс: Комп'ютерні мережі | 2-3 курс
 
---
 
## 1 Як це працює
 
Маршрутизатор підключений до двох провайдерів одночасно. У нормальному режимі весь трафік іде через **основний (primary)** канал. Якщо він падає - трафік автоматично перемикається на **резервний (backup)** канал. Повернення на основний відбувається також автоматично - без втручання адміністратора
 
Механізм реалізується через три компоненти:
 
```
IP SLA          → постійно перевіряє доступність каналу (ICMP probe)
Object Tracking → слідкує за результатом SLA і змінює стан track (up/down)
Floating Static Route → основний маршрут прив'язаний до track,
                         резервний має вищу адміністративну відстань
```
 
**Логіка перемикання:**
 
```
SLA probe успішний → track 1 = UP   → активний маршрут через ISP1 (primary)
SLA probe провалився → track 1 = DOWN → маршрут через ISP1 видаляється з таблиці
                                       → активується маршрут через ISP2 (backup)
```
 
---

## 2 Схема мережі
 
> 📁 Зображення: `../assets/network-configs_isp-failover_1.svg`
 
```
                      ICMP probe → 1.0.0.1
                           │
[ISP 1 — Primary]          │         [ISP 2 — Backup]
 GW: 203.0.113.97          │          GW: 192.0.2.254
        │                  │                  │
        │ Et0/1            │         Et0/0    │
        │ 203.0.113.98     │    192.0.2.21    │
        └──────────[   Cisco Router   ]───────┘
                           │ Et0/2
                           │ 192.168.50.1
                      [LAN 192.168.50.0/24]
```
 
| Інтерфейс | Опис | Адреса |
|-----------|------|--------|
| `Ethernet0/1` | ISP 1 — Primary | 203.0.113.98/30 |
| `Ethernet0/0` | ISP 2 — Backup | 192.0.2.21/24 |
| `Ethernet0/2` | LAN | 192.168.50.1/24 |
 
---
 
## 3 Повна конфігурація

 
### 3.1 Налаштування інтерфейсів
 
```
! ISP 1 - основний канал
interface Ethernet0/1
 description ISP_1_PRIMARY
 ip address 203.0.113.98 255.255.255.252
 ip nat outside
 no shutdown
 
! ISP 2 - резервний канал
interface Ethernet0/0
 description ISP_2_BACKUP
 ip address 192.0.2.21 255.255.255.0
 ip nat outside
 no shutdown
 
! LAN - внутрішня мережа
interface Ethernet0/2
 description LOCAL_LAN
 ip address 192.168.50.1 255.255.255.0
 ip nat inside
 no shutdown
```

---
 
### 3.2 Налаштування IP SLA
 
IP SLA - це механізм активного моніторингу. Маршрутизатор сам генерує ICMP-запити і відстежує відповіді
 
```
ip sla 1
 icmp-echo 1.0.0.1 source-interface Ethernet0/1
 threshold 4
 timeout 10000
 frequency 30
ip sla schedule 1 life forever start-time now
```
 
**Опис параметрів:**
 
| Параметр | Значення | Опис |
|----------|----------|------|
| `ip sla 1` | — | Номер SLA-операції (1–2147483647) |
| `icmp-echo 1.0.0.1` | IP-адреса цілі | Хост до якого надсилається ICMP. Має бути за межами маршрутизатора у боці ISP1. `1.0.0.1` — публічна адреса Cloudflare (надійна ціль) |
| `source-interface Ethernet0/1` | Інтерфейс ISP1 | Зондування відбувається саме через primary-канал, тому падіння ISP1 одразу вимикає SLA |
| `threshold 4` | мс | Якщо RTT > 4 мс — вважати відповідь як "за межами порогу" (впливає на статистику, не на reachability) |
| `timeout 10000` | мс (10 сек) | Час очікування відповіді. Якщо відповідь не прийшла за 10 секунд — probe вважається невдалим |
| `frequency 30` | сек | Інтервал між probe-запитами. Кожні 30 секунд надсилається новий ICMP |
| `life forever` | — | SLA працює постійно, без обмеження часу |
| `start-time now` | — | Почати негайно після застосування конфігурації |
 
!!! warning "Чому саме 1.0.0.1, а не шлюз ISP?"
    Якщо як ціль вказати шлюз провайдера `203.0.113.97` — SLA буде `up` навіть якщо за шлюзом немає Інтернету (наприклад, проблеми у провайдера далі по мережі). Адреса `1.0.0.1` (Cloudflare) перевіряє реальну досяжність Інтернету наскрізь
 
---
 
### 3.3 Налаштування Object Tracking
 
Track слідкує за станом SLA і перетворює його в об'єкт `up/down` до якого можна прив'язати маршрут
 
```
track 1 ip sla 1 reachability
 delay down 10 up 10
```
 
**Опис параметрів:**
 
| Параметр | Значення | Опис |
|----------|----------|------|
| `track 1` | — | Номер об'єкта відстеження |
| `ip sla 1 reachability` | — | Тип відстеження — досяжність (чи приходить відповідь на SLA probe) |
| `delay down 10` | сек | Перш ніж оголосити track DOWN — чекати 10 сек. Захист від миттєвого перемикання при одному втраченому probe |
| `delay up 10` | сек | Перш ніж оголосити track UP після відновлення — чекати 10 сек. Захист від флапінгу при нестабільному каналі |
 
!!! tip "Навіщо delay?"
    Без `delay` одна загублена відповідь на probe одразу перемикає маршрут. `delay down 10` означає що канал має бути недоступний **мінімум 10 секунд** перед перемиканням. Це захищає від хибних спрацьовувань через тимчасові втрати пакетів
 
---
 
### 3.4 Налаштування маршрутів (floating static routes)
 
```
! Маршрут до цілі SLA через ISP1 (потрібен щоб probe міг вийти назовні)
ip route 1.0.0.1 255.255.255.255 203.0.113.97
 
! Основний маршрут за замовчуванням через ISP1 - прив'язаний до track 1
! Якщо track 1 = DOWN - цей маршрут видаляється з таблиці маршрутизації
ip route 0.0.0.0 0.0.0.0 203.0.113.97 track 1
 
! Резервний маршрут через ISP2 - адміністративна відстань 10
! AD=10 > AD=1 (статичний), тому активується лише коли основний маршрут відсутній
ip route 0.0.0.0 0.0.0.0 192.0.2.254 10
```
 
!!! info "Адміністративна відстань (AD)"
    Cisco IOS обирає маршрут з меншою AD. Статичний маршрут має AD=1. Якщо додати `10` в кінці команди `ip route` - AD стає 11. Поки основний маршрут (AD=1) присутній у таблиці - резервний (AD=11) ігнорується. Як тільки основний видаляється (track DOWN) - резервний стає активним
 
---
 
### 3.5 Налаштування NAT (per-ISP)
 
При переключенні між провайдерами потрібно щоб NAT також використовував відповідний інтерфейс. Без цього пакети будуть виходити через ISP2, але з адресою ISP1 - і відповіді не повернуться
 
```
! ACL - які підмережі підлягають NAT трансляції
access-list 1 permit 192.168.50.0 0.0.0.255
! (потрібно додати всі внутрішні підмережі що мають виходити в Інтернет)
 
! Route-map для ISP1 - NAT через Et0/1
route-map RM_ISP1 permit 10
 match ip address 1
 match interface Ethernet0/1
 
! Route-map для ISP2 - NAT через Et0/0
route-map RM_ISP2 permit 10
 match ip address 1
 match interface Ethernet0/0
 
! Прив'язати NAT до route-map
ip nat inside source route-map RM_ISP1 interface Ethernet0/1 overload
ip nat inside source route-map RM_ISP2 interface Ethernet0/0 overload
```
 
!!! warning "Без route-map NAT не переключається автоматично"
    Стандартний `ip nat inside source list 1 interface Et0/1 overload` прив'язаний до конкретного інтерфейсу жорстко. При переключенні на ISP2 трафік буде маршрутизуватись правильно, але NAT трансляція залишиться зі старою адресою. Route-map вирішує це - він вибирає правило NAT залежно від того через який інтерфейс іде пакет
 
---
 
## 4 Перевірка та діагностика
 
```
! Стан SLA операцій
show ip sla summary
show ip sla statistics 1
show ip sla statistics 1 details
 
! Стан object tracking
show track
show track 1
show track brief
 
! Таблиця маршрутизації (перевірити який маршрут активний)
show ip route
show ip route 0.0.0.0
show ip route static
 
! Таблиця NAT трансляцій
show ip nat translations
show ip nat statistics
 
! Перевірка CEF
show ip cef
show ip cef 0.0.0.0
```
 
### 4.1 Приклад виводу show ip sla statistics 1
 
```
Router# show ip sla statistics 1
IPSLAs Latest Operation Statistics
 
IPSLA operation id: 1
        Latest RTT: 3 milliseconds
Latest operation start time: 12:00:00 UTC
Latest operation return code: OK        ← probe успішний
Number of successes: 1440
Number of failures: 2
Operation time to live: Forever
```
 
### 4.2 Приклад виводу show track 1
 
```
Router# show track 1
Track 1
  IP SLA 1 Reachability
  Reachability is Up           ← стан каналу
    2 changes, last change 00:14:32
  Delay up 10 secs, down 10 secs
  Latest operation return code: OK
  Latest RTT (millisecs) 3
  Tracked by:
    Static IP Routing 0        ← прив'язаний маршрут
```
 
---
 
## 5 Debug
 
```
! Debug SLA probe (показує кожен запит та відповідь)
debug ip sla trace
debug ip sla error
 
! Debug object tracking
debug track
 
! Debug маршрутизації (зміни в таблиці)
debug ip routing
 
! Зупинити debug
no debug all
undebug all
```
 
!!! danger "Обережно з debug на завантажених маршрутизаторах"
    `debug ip sla trace` з `frequency 30` генерує записи кожні 30 секунд - це прийнятно. `debug ip routing` може давати дуже багато виводу при нестабільній мережі. Завжди завершуй `no debug all`
 
---
 
## 6 Опціональні конфігурації
 
### 6.1 Зміна таймерів під різні ситуації
 
```
! Агресивний режим - швидке перемикання (нестабільний ISP)
ip sla 1
 icmp-echo 1.0.0.1 source-interface Ethernet0/1
 timeout 2000        ! 2 сек — швидке виявлення
 frequency 5         ! кожні 5 сек — часте зондування
ip sla schedule 1 life forever start-time now
 
track 1 ip sla 1 reachability
 delay down 3 up 10  ! вниз - швидко, вгору - обережно
 
! Консервативний режим - повільне перемикання (стабільний ISP, чутливо до флапів)
ip sla 1
 icmp-echo 1.0.0.1 source-interface Ethernet0/1
 timeout 15000       ! 15 сек
 frequency 60        ! кожну хвилину
 
track 1 ip sla 1 reachability
 delay down 60 up 120 ! вниз - хвилина, вгору - дві хвилини
```
 
### 6.2 Три провайдери (ISP1 primary, ISP2 secondary, ISP3 tertiary)
 
```
! SLA для ISP1 і ISP2
ip sla 1
 icmp-echo 1.0.0.1 source-interface Ethernet0/1
 timeout 10000
 frequency 30
ip sla schedule 1 life forever start-time now
 
ip sla 2
 icmp-echo 1.0.0.1 source-interface Ethernet0/0
 timeout 10000
 frequency 30
ip sla schedule 2 life forever start-time now
 
! Track для кожного SLA
track 1 ip sla 1 reachability
 delay down 10 up 10
 
track 2 ip sla 2 reachability
 delay down 10 up 10
 
! Три маршрути з різними AD
ip route 0.0.0.0 0.0.0.0 203.0.113.97 track 1       ! ISP1, AD=1
ip route 0.0.0.0 0.0.0.0 192.0.2.254  10 track 2    ! ISP2, AD=11
ip route 0.0.0.0 0.0.0.0 10.10.10.1   20             ! ISP3, AD=21 (завжди резерв)
 
! SLA probe для ISP2 через ISP2
ip route 1.0.0.1 255.255.255.255 203.0.113.97        ! probe ISP1
ip route 8.8.8.8 255.255.255.255 192.0.2.254         ! probe ISP2 (інша ціль)
```
 
### 6.3 Використання декількох цілей для надійнішої діагностики (SLA OR-логіка)
 
```
! Два SLA на різні цілі через один інтерфейс
ip sla 1
 icmp-echo 1.0.0.1 source-interface Ethernet0/1
 timeout 10000
 frequency 30
ip sla schedule 1 life forever start-time now
 
ip sla 2
 icmp-echo 8.8.8.8 source-interface Ethernet0/1
 timeout 10000
 frequency 30
ip sla schedule 2 life forever start-time now
 
! Boolean track - ISP1 вважається DOWN якщо ОБИДВІ цілі недоступні
track 10 list boolean or
 object 1
 object 2
 delay down 10 up 10
 
ip route 0.0.0.0 0.0.0.0 203.0.113.97 track 10
ip route 0.0.0.0 0.0.0.0 192.0.2.254 10
```
 
!!! tip "Boolean OR vs AND"
    `boolean or` - перемикання якщо хоч одна ціль недоступна (агресивно)
    `boolean and` - перемикання тільки якщо обидві цілі недоступні (консервативно, захист від хибних спрацьовувань)
 
### 6.4 Очищення NAT-таблиці при переключенні
 
При переключенні між провайдерами активні NAT-сесії залишаються зі старими трансляціями — нові пакети підуть через ISP2, але відповіді на старі сесії не прийдуть. Можна очистити таблицю вручну або через EEM (Embedded Event Manager):
 
```
! Вручну після переключення
clear ip nat translation *
 
! Через EEM — автоматично при зміні стану track
event manager applet CLEAR-NAT-ON-FAILOVER
 event track 1 state down
 action 1.0 cli command "enable"
 action 2.0 cli command "clear ip nat translation *"
 action 3.0 syslog msg "ISP1 DOWN - NAT table cleared"
```
 
---
 
> 📌 **Зберегти конфігурацію:** `copy running-config startup-config` або `write memory`
 
---
 
!!! quote "Джерело"
    Стаття базується на офіційній документації Cisco та реальній конфігурації кафедри
    Оригінал (англійською): [Configuring IP SLAs ICMP Echo Operations](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipsla/configuration/15-mt/sla-15-mt-book/sla_icmp_echo.html)