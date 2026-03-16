# DHCP-сервер на Cisco IOS — Шпаргалка

> Курс: Комп'ютерні мережі | 2 курс

---

## 1 Протокол DHCP

**DHCP (Dynamic Host Configuration Protocol)** — протокол автоматичного призначення мережевих параметрів клієнтам. Замість того щоб вручну налаштовувати IP-адресу, маску, шлюз та DNS на кожному пристрої — маршрутизатор або сервер видає ці параметри автоматично при підключенні

Що видає DHCP-сервер клієнту:

- IP-адресу та маску підмережі
- Шлюз за замовчуванням (default gateway)
- DNS-сервери
- Доменне ім'я
- Час оренди адреси (lease time)
- Додаткові опції (TFTP-сервер, VoIP, тощо)

### 1.1 Процес DORA

Отримання адреси відбувається у чотири кроки — **D**iscover → **O**ffer → **R**equest → **A**cknowledge:

```
Клієнт                                          DHCP-сервер
  |                                                   |
  |──① DHCPDISCOVER──────────────────────────────────►|
  |   Broadcast: "Хто може дати мені адресу?"         |
  |                                                   |
  |◄─② DHCPOFFER──────────────────────────────────────|
  |   "Пропоную: 192.168.1.100/24, GW: 192.168.1.1"   |
  |                                                   |
  |──③ DHCPREQUEST───────────────────────────────────►|
  |   Broadcast: "Беру адресу 192.168.1.100"          |
  |   (щоб інші сервери знали про вибір)              |
  |                                                   |
  |◄─④ DHCPACK────────────────────────────────────────|
  |   "Підтверджую. Оренда: 8 годин"                  |
  |                                                   |
```

!!! info "Чому DHCPREQUEST — broadcast?"
    На кроці ③ клієнт ще не має IP-адреси. Broadcast-повідомлення дозволяє повідомити **всі** DHCP-сервери в мережі про те, яку пропозицію було прийнято — решта серверів відкликають свої оферти

---

## 2 Базове налаштування DHCP-сервера

### 2.1 Схема мережі для прикладу

```
[PC-1]──┐
[PC-2]──┤──[Switch]──[fa0/0: 192.168.1.1/24]──[Router]──[Internet]
[PC-3]──┘                                       DHCP-сервер
                                                Пул: 192.168.1.0/24
```

### 2.2 Крок 1 — Виключити статичні адреси з пулу

Перш за все потрібно зарезервувати адреси, які **не** повинні видаватися клієнтам, їх статично можна прописати на: шлюзі, принтерах, серверах, тощо

```
! Виключити діапазон (шлюз та статичні пристрої)
ip dhcp excluded-address 192.168.1.1 192.168.1.10

! Виключити одну адресу
ip dhcp excluded-address 192.168.1.254
```

### 2.3 Крок 2 — Налаштувати пул адрес

```
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0    ! підмережа для роздачі
 default-router 192.168.1.1           ! шлюз за замовчуванням
 dns-server 8.8.8.8 1.1.1.1           ! DNS-сервери (до 8 адрес)
 domain-name example.local            ! доменне ім'я
 lease 8                              ! оренда 8 днів (default: 1 день)
```

### 2.4 Крок 3 — Перевірити що сервіс увімкнено

```
! DHCP-сервер увімкнено за замовчуванням
! Якщо раніше вимикали — увімкнути назад:
service dhcp

! Переконатись що рядка "no service dhcp" немає у конфізі:
show running-config | include service dhcp
```

### 2.5 Повний приклад базового налаштування

```
! --- Виключити статичні адреси ---
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.1.254

! --- Пул для LAN ---
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8 8.8.4.4
 domain-name company.local
 lease 8
 exit

! --- Інтерфейс маршрутизатора (шлюз для клієнтів) ---
interface FastEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
```

---

## 3 DHCP Relay - сервер в іншій підмережі

Якщо DHCP-сервер знаходиться в іншій підмережі, маршрутизатор повинен пересилати DHCP-запити до нього за допомогою **ip helper-address**

```
                          ip helper-address 10.0.0.10
[Client: 192.168.1.x]──[fa0/0: 192.168.1.1]──[Router]──[DHCP-сервер: 10.0.0.10]
```

```
! На інтерфейсі, що дивиться до клієнтів:
interface FastEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip helper-address 10.0.0.10          ! адреса DHCP-сервера
 no shutdown
```

!!! tip "Декілька серверів"
    Можна вказати кілька `ip helper-address` для резервування — маршрутизатор надсилатиме запит на всі адреси одночасно

---

## 4 Перевірка та діагностика

```
! Переглянути таблицю виданих адрес
show ip dhcp binding
show ip dhcp binding 192.168.1.100    ! конкретна адреса

! Статистика сервера (кількість пакетів DISCOVER/OFFER/REQUEST/ACK)
show ip dhcp server statistics

! Переглянути конфлікти адрес
show ip dhcp conflict

! Переглянути налаштовані пули
show ip dhcp pool

! Очистити таблицю виданих адрес
clear ip dhcp binding *               ! всі записи
clear ip dhcp binding 192.168.1.100  ! конкретний запис

! Очистити лічильники конфліктів
clear ip dhcp conflict *

! Debug
debug ip dhcp server events           ! події DHCP-сервера
debug ip dhcp server packets          ! пакети
no debug all
```

### 4.1 Приклад виводу show ip dhcp binding

```
Router# show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address      Client-ID/        Lease expiration        Type       State      Interface
                Hardware address/
                User name
192.168.1.11    0100.5056.c000.01  Mar 24 2026 09:00 AM   Automatic  Active     Fa0/0
192.168.1.12    0100.5056.c000.02  Mar 24 2026 11:30 AM   Automatic  Active     Fa0/0
192.168.1.100   0100.50ab.1234.56  infinite               Manual     Active     Fa0/0
```

---

## 5 Резервування адреси по MAC (Static Binding)

Дозволяє закріпити конкретну IP-адресу за пристроєм на основі його MAC-адреси. Корисно для принтерів, серверів, IP-камер — пристроїв що мають отримувати адресу через DHCP але завжди одну і ту ж

```
! Кожне резервування — окремий пул типу host
ip dhcp pool PRINTER-01
 host 192.168.1.200 255.255.255.0     ! зарезервована адреса
 hardware-address 00e0.4c68.1234      ! MAC-адреса пристрою
 default-router 192.168.1.1
 dns-server 8.8.8.8
 client-name printer-01               ! необов'язково — ім'я для зручності
 exit

ip dhcp pool CAMERA-01
 host 192.168.1.201 255.255.255.0
 hardware-address b4a9.fc00.aabb
 default-router 192.168.1.1
 exit
```

!!! warning "Окремий пул для кожного пристрою"
    Статичне резервування **не можна** змішувати з динамічним пулом (`network`). Для кожного зарезервованого пристрою потрібен окремий `ip dhcp pool`

!!! info "Де знайти MAC-адресу?"
    На клієнті Windows: `ipconfig /all` → Physical Address
    На Linux/Mac: `ip link show` або `ifconfig`
    На Cisco: `show arp` або `show mac address-table`

---

## 6 Додаткові DHCP-опції

DHCP може передавати клієнтам додаткові параметри через **опції**. Деякі задаються іменованими командами, інші — через універсальну команду `option <номер>`

### 6.1 Стандартні опції через іменовані команди

```
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1           ! опція 3
 dns-server 8.8.8.8 8.8.4.4           ! опція 6
 domain-name company.local            ! опція 15
 netbios-name-server 192.168.1.5      ! опція 44 — WINS-сервер (для Windows)
 netbios-node-type h-node             ! опція 46 — тип NetBIOS-вузла
 lease 7                              ! опція 51 — час оренди в днях
```

### 6.2 Опція 43 — Vendor Specific (Cisco WLC / точки доступу)

Використовується для передачі адреси Cisco Wireless LAN Controller точкам доступу (Cisco AP)

```
ip dhcp pool AP-POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 ! Опція 43 — адреса WLC у hex форматі
 ! Формат: f1:04:<IP у hex>
 ! 192.168.10.200 = c0.a8.0a.c8
 option 43 hex f104c0a80ac8
```

### 6.3 Опція 66 / 67 — TFTP-сервер та ім'я файлу (PXE Boot, VoIP)

Використовується для мережевого завантаження (PXE) або автоматичного оновлення прошивки IP-телефонів

```
ip dhcp pool VOIP-POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
 option 66 ascii 192.168.20.10        ! адреса TFTP-сервера (next-server)
 option 67 ascii SEP.cnf.xml          ! ім'я конфігураційного файлу
 ! Або через окремі команди:
 next-server 192.168.20.10            ! аналог опції 66
 bootfile SEP.cnf.xml                 ! аналог опції 67
```

### 6.4 Опція 138 — CAPWAP / Cisco AP Controller

Альтернативний метод передачі адреси WLC для новіших точок доступу Cisco (замість опції 43)

```
ip dhcp pool AP-POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 ! Опція 138 — IP-адреса WLC (підтримує до 4 адрес контролерів)
 option 138 ip 192.168.100.10
 ! Або декілька WLC через пробіл:
 option 138 ip 192.168.100.10 192.168.100.11
```

### 6.5 Опція 150 — TFTP для Cisco IP-телефонів

Cisco IP Phone використовує опцію 150 (не стандартну 66) для отримання адреси TFTP-сервера з конфігурацією

```
ip dhcp pool PHONE-POOL
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
 option 150 ip 192.168.30.5           ! адреса TFTP-сервера для IP-телефонів
 lease 1
```

### 6.6 Таблиця популярних DHCP-опцій

| Опція | Назва | Команда / Застосування |
|-------|-------|----------------------|
| 3 | Default Gateway | `default-router` |
| 6 | DNS Servers | `dns-server` |
| 15 | Domain Name | `domain-name` |
| 43 | Vendor Specific | `option 43 hex ...` — Cisco WLC (старіші AP) |
| 44 | NetBIOS Name Server | `netbios-name-server` — WINS |
| 46 | NetBIOS Node Type | `netbios-node-type` |
| 51 | Lease Time | `lease` |
| 66 | TFTP Server Name | `option 66 ascii` або `next-server` |
| 67 | Bootfile Name | `option 67 ascii` або `bootfile` |
| 82 | Relay Agent Info | Автоматично — relay agent |
| 138 | CAPWAP AC Address | `option 138 ip` — Cisco WLC (нові AP) |
| 150 | TFTP Server | `option 150 ip` — Cisco IP Phone |

---

## 7 Збереження бази адрес (Database Agent)

За замовчуванням таблиця виданих адрес зберігається в оперативній пам'яті і **втрачається при перезавантаженні**. Щоб уникнути конфліктів адрес після ребуту можна налаштувати збереження на TFTP/FTP

```
! Зберігати на TFTP-сервері кожні 300 секунд
ip dhcp database tftp://192.168.1.10/dhcp-bindings.txt timeout 300 write-delay 60

! Зберігати на FTP
ip dhcp database ftp://admin:password@192.168.1.10/dhcp-bindings.txt

! Або - вимкнути логування конфліктів (якщо Database Agent не використовується)
no ip dhcp conflict logging
```

---

## 8 Кілька пулів на одному маршрутизаторі

Маршрутизатор може обслуговувати кілька підмереж - наприклад, якщо є кілька VLAN або інтерфейсів

```
! Пул для VLAN 10 (співробітники)
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool VLAN10-STAFF
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 lease 8

! Пул для VLAN 20 (гостьова мережа)
ip dhcp excluded-address 192.168.20.1 192.168.20.5
ip dhcp pool VLAN20-GUEST
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8 1.1.1.1
 lease 1                              ! гостям — коротша оренда

! Підінтерфейси для VLAN (Router-on-a-Stick)
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

---

> 📌 **Зберегти конфігурацію:** `copy running-config startup-config` або `wr`

---

!!! quote "Джерело"
    Деякі матеріали взято офіційної документації Cisco
    Оригінал (англійською): [Configuring the Cisco IOS DHCP Server](https://www.cisco.com/en/US/docs/ios/12_4t/ip_addr/configuration/guide/htdhcpsv.html)
