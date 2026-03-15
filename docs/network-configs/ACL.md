# IP Access Control Lists — Шпаргалка

## 1 Основи ACL

### 1.1 Що таке ACL

ACL (Access Control List) — це набір правил, що дозволяють або забороняють проходження мережевого трафіку через інтерфейс маршрутизатора. Правила перевіряються **по порядку зверху вниз** до першого збігу. Якщо жодне правило не спрацювало — трафік **блокується** (неявний `deny any` в кінці кожного списку).

```
! Два рівнозначних ACL — з явним і неявним deny:
access-list 101 permit ip 10.1.1.0 0.0.0.255 172.16.1.0 0.0.0.255

access-list 102 permit ip 10.1.1.0 0.0.0.255 172.16.1.0 0.0.0.255
access-list 102 deny ip any any          ! явний deny (результат той самий)
```

!!! warning "Важливо"
    В кожному ACL має бути хоча б один `permit`, інакше весь трафік буде заблокований.

### 1.2 Wildcard-маска (інверсна маска)

На відміну від звичайної маски підмережі, в ACL використовується **wildcard-маска**:

- `0` — біт **повинен** збігатися
- `1` — біт **ігнорується**

```
! Формула: 255.255.255.255 - маска_підмережі = wildcard
! Приклад для мережі 192.168.1.0/24:
! 255.255.255.255 - 255.255.255.0 = 0.0.0.255

access-list 1 permit 192.168.1.0 0.0.0.255   ! вся мережа /24

! Скорочення:
! 0.0.0.0 255.255.255.255 = any    (будь-яка адреса)
! host 10.1.1.2            = 10.1.1.2 0.0.0.0  (конкретний хост)
```

### 1.3 Напрямки застосування ACL

| Напрямок | Опис |
|----------|------|
| `in` | Перевірка пакетів **при вході** на інтерфейс (ще до маршрутизації) |
| `out` | Перевірка пакетів **при виході** з інтерфейсу (після маршрутизації) |

```
! ACL краще застосовувати на інтерфейсі, найближчому до джерела трафіку

interface GigabitEthernet0/0
 ip access-group 101 in    ! застосувати ACL 101 на вхід
 ip access-group 102 out   ! застосувати ACL 102 на вихід
```

---

## 2 Стандартні ACL

Фільтрують трафік **тільки за IP-адресою джерела**. Номери: `1–99` та `1300–1999`.

### 2.1 Синтаксис та приклади

```
! Синтаксис:
access-list <номер> {permit|deny} {host <ip> | <мережа> <wildcard> | any}

! Дозволити трафік з мережі 10.1.1.0/24
access-list 1 permit 10.1.1.0 0.0.0.255

! Дозволити трафік тільки від конкретного хоста
access-list 1 permit host 10.1.1.2

! Заблокувати конкретний хост, решту дозволити
access-list 2 deny host 192.168.1.100
access-list 2 permit any
```

### 2.2 Застосування до інтерфейсу

```
interface Ethernet0/0
 ip address 10.1.1.1 255.255.255.0
 ip access-group 1 in

! Стандартний ACL — застосовувати БЛИЖЧЕ ДО ПРИЗНАЧЕННЯ,
! бо фільтрує тільки за джерелом і може ненавмисно
! заблокувати трафік до інших вузлів
```

---

## 3 Розширені ACL

Фільтрують за **IP-адресою джерела і призначення**, протоколом, портами. Номери: `100–199` та `2000–2699`.

### 3.1 Синтаксис

```
! IP
access-list <номер> {permit|deny} ip <src> <wildcard> <dst> <wildcard>

! TCP/UDP
access-list <номер> {permit|deny} tcp <src> <wildcard> <dst> <wildcard> eq <порт>

! ICMP
access-list <номер> {permit|deny} icmp <src> <wildcard> <dst> <wildcard> [тип]
```

### 3.2 Приклади

```
! Дозволити HTTP та HTTPS з внутрішньої мережі назовні
access-list 101 permit tcp 10.1.1.0 0.0.0.255 any eq 80
access-list 101 permit tcp 10.1.1.0 0.0.0.255 any eq 443

! Дозволити SSH тільки з адміністративної підмережі
access-list 101 permit tcp 192.168.100.0 0.0.0.255 any eq 22

! Заблокувати ping ззовні, але дозволити відповіді на наші ping
access-list 101 deny icmp any 10.1.1.0 0.0.0.255 echo
access-list 101 permit ip any 10.1.1.0 0.0.0.255

! Дозволити Telnet тільки між двома конкретними хостами
access-list 101 permit tcp host 10.1.1.2 host 172.16.1.1 eq telnet

! Розширений ACL — застосовувати БЛИЖЧЕ ДО ДЖЕРЕЛА трафіку
interface Ethernet0/1
 ip address 172.16.1.2 255.255.255.0
 ip access-group 101 in
```

### 3.3 Ключові слова для портів

```
! Оператори порівняння портів:
! eq  — дорівнює (equal)
! neq — не дорівнює (not equal)
! gt  — більше (greater than)
! lt  — менше (less than)
! range — діапазон

access-list 102 permit tcp any any eq 80          ! тільки HTTP
access-list 102 permit tcp any any eq 443         ! тільки HTTPS
access-list 102 permit tcp any any range 1024 65535
access-list 102 deny tcp any any lt 1024          ! блокувати well-known порти

! Добре відомі порти (можна використовувати ім'я замість номера):
! ftp(21), ssh(22), telnet(23), smtp(25), dns(53), http(80),
! https(443), ntp(123), snmp(161), syslog(514)
access-list 102 permit tcp any any eq www         ! те саме що eq 80
```

---

## 4 Іменовані ACL

Замість номерів використовуються **імена** — зручніше редагувати та розуміти призначення.

### 4.1 Синтаксис та приклади

```
! Стандартний іменований ACL
ip access-list standard BLOCK-GUEST
 deny host 10.0.0.100
 permit 10.0.0.0 0.0.0.255
 deny any

! Розширений іменований ACL
ip access-list extended PERMIT-WEB-ONLY
 permit tcp 10.1.1.0 0.0.0.255 any eq 80
 permit tcp 10.1.1.0 0.0.0.255 any eq 443
 permit icmp 10.1.1.0 0.0.0.255 any echo
 deny ip any any

! Застосування
interface GigabitEthernet0/0
 ip access-group PERMIT-WEB-ONLY in
```

### 4.2 Редагування іменованих ACL

```
! Перегляд ACL з порядковими номерами рядків
show ip access-lists PERMIT-WEB-ONLY

! Видалити конкретний рядок
ip access-list extended PERMIT-WEB-ONLY
 no permit icmp 10.1.1.0 0.0.0.255 any echo

! Додати рядок між існуючими (за порядковим номером)
ip access-list extended PERMIT-WEB-ONLY
 15 permit tcp 10.1.1.0 0.0.0.255 any eq 8080
```

!!! info "Редагування нумерованих ACL"
    У нумерованих ACL команда `no access-list 101 deny icmp any any` видаляє **весь список**, а не один рядок. Для редагування окремих рядків використовуй іменовані ACL або порядкові номери рядків через `ip access-list extended 101`.

---

## 5 Порядкові номери рядків ACL

```
! Додати рядок у конкретну позицію нумерованого ACL
Router(config)#ip access-list extended 101
Router(config-ext-nacl)#5 deny tcp any any eq telnet   ! вставити першим

show access-list
! Extended IP access list 101
!     5 deny tcp any any eq telnet     ← новий рядок на початку
!     10 permit tcp any any
!     20 permit udp any any
!     30 permit icmp any any

! Приклад вставки між рядками
ip access-list extended 101
 18 permit tcp any host 172.16.2.11    ! між рядками 15 і 20
```

---

## 6 Часові ACL (Time-Based)

Дозволяють застосовувати правила тільки у визначений час.

```
! Крок 1 — Визначити часовий діапазон
time-range BUSINESS-HOURS
 periodic Monday Wednesday Friday 8:00 to 17:00

time-range WORK-WEEK
 periodic weekdays 9:00 to 18:00

time-range ONE-TIME
 absolute start 00:00 01 Jan 2025 end 23:59 31 Dec 2025

! Крок 2 — Застосувати до ACL
access-list 101 permit tcp 10.1.1.0 0.0.0.255 172.16.1.0 0.0.0.255 eq telnet time-range BUSINESS-HOURS

! Перевірка
show time-range
```

!!! tip "Синхронізація часу"
    Для коректної роботи часових ACL налаштуй NTP: `ntp server 0.pool.ntp.org`

---

## 7 Рефлексивні ACL

Автоматично дозволяють зворотний трафік для сесій, ініційованих зсередини. Аналог stateful firewall.

```
! Крок 1 — ACL для вихідного трафіку (reflect створює динамічний запис)
ip access-list extended OUTBOUND-FILTERS
 permit icmp 10.1.1.0 0.0.0.255 172.16.1.0 0.0.0.255
 permit tcp 10.1.1.0 0.0.0.255 172.16.1.0 0.0.0.255 reflect TCP-TRAFFIC

! Крок 2 — ACL для вхідного трафіку (evaluate перевіряє динамічні записи)
ip access-list extended INBOUND-FILTERS
 permit icmp 172.16.1.0 0.0.0.255 10.1.1.0 0.0.0.255
 evaluate TCP-TRAFFIC

! Крок 3 — Застосування до інтерфейсу
interface Ethernet0/1
 ip address 172.16.1.2 255.255.255.0
 ip access-group INBOUND-FILTERS in
 ip access-group OUTBOUND-FILTERS out

! Таймаут для рефлексивних записів (за замовчуванням 300 сек)
ip reflexive-list timeout 120
```

---

## 8 Коментарі в ACL

```
! Коментарі до нумерованого ACL
access-list 101 remark === ДОЗВОЛИТИ WEB-ТРАФІК ===
access-list 101 permit tcp 10.1.1.0 0.0.0.255 any eq 80
access-list 101 remark --- Блокувати решту ---
access-list 101 deny ip any any

! Коментарі до іменованого ACL
ip access-list extended INTERNET-ACCESS
 remark Дозволити HTTP та HTTPS
 permit tcp 10.0.0.0 0.0.0.255 any eq 80
 permit tcp 10.0.0.0 0.0.0.255 any eq 443
 remark Дозволити DNS
 permit udp 10.0.0.0 0.0.0.255 any eq 53
 remark Заблокувати решту
 deny ip any any
```

---

## 9 Підсумок мереж для ACL

Оптимізація: замість кількох рядків — один узагальнений запис.

```
! 8 мереж від 192.168.32.0/24 до 192.168.39.0/24
! можна замінити одним рядком:
access-list 10 permit 192.168.32.0 0.0.7.255    ! покриває .32–.39

! 2 мережі: 192.168.146.0/24 та 192.168.147.0/24 → /23
access-list 10 permit 192.168.146.0 0.0.1.255

! 2 мережі: 192.168.148.0/24 та 192.168.149.0/24 → /23
access-list 10 permit 192.168.148.0 0.0.1.255
```

---

## 10 Перевірка та діагностика

```
! Переглянути всі ACL та лічильники спрацювань
show access-lists
show ip access-lists
show ip access-lists 101
show ip access-lists PERMIT-WEB-ONLY

! Переглянути ACL застосовані до інтерфейсів
show ip interface GigabitEthernet0/0
show ip interface brief

! Видалити ACL з інтерфейсу
interface GigabitEthernet0/0
 no ip access-group 101 in

! Видалити весь нумерований ACL
no access-list 101

! Debug пакетів (обережно на завантажених пристроях!)
access-list 101 permit ip any host 10.2.6.6
access-list 101 permit ip host 10.2.6.6 any

interface GigabitEthernet0/0
 no ip route-cache              ! вимкнути fast switching для debug

debug ip packet 101             ! debug з фільтром ACL
debug ip packet 101 detail      ! детальний вивід
no debug all                    ! зупинити debug

interface GigabitEthernet0/0
 ip route-cache                 ! увімкнути fast switching назад

! Перевірка логів (з ключовим словом log в ACL)
access-list 101 deny ip any any log
show logging
```

!!! danger "Увага при debug"
    Команда `debug ip packet` без фільтра ACL може перевантажити CPU маршрутизатора. Завжди використовуй ACL-фільтр як у прикладі вище.

---

> 📌 **Зберегти конфігурацію:** `copy running-config startup-config` або `wr`

---

!!! quote "Джерело"
    Стаття базується на офіційній документації Cisco TAC.
    Оригінал (англійською): [Configure IP Access Lists](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html)