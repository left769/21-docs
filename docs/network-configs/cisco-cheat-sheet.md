# Cisco IOS — Шпаргалка команд
> Курс: Комп'ютерні мережі | 2 курс

---

## Зміст
- [1 Device Fundamentals](#1-device-fundamentals)
- [2 Навігація та основи IOS](#2-навігація-та-основи-ios)
- [4 File System Operations](#4-file-system-operations)
- [5 Licensing Management](#5-licensing-management)
- [6 Device Management](#6-device-management)
- [7 Security Fundamentals](#7-security-fundamentals)
- [8 AAA & Access Control](#8-aaa--access-control)
- [9 Management Protocols](#9-management-protocols)
- [10 Ethernet Switching](#10-ethernet-switching)
- [11 VLANs & Trunking](#11-vlans--trunking)
- [12 Spanning Tree Protocol](#12-spanning-tree-protocol)
- [13 EtherChannel & LACP](#13-etherchannel--lacp)
- [14 IP Addressing & Services](#14-ip-addressing--services)
- [15 Static Routing](#15-static-routing)

---

## 1 Основи роботи

### 1.1 Скидання налаштувань

> ⚠️ Ці команди повністю видалять дані з пристрою. Завжди зберігайте резервні копії!

```
! Повне скидання (знищення всіх даних)
Router# delete nvram:startup-config
Router# delete flash:vlan.dat
Router# erase nvram:
Router# erase flash:
Router# write erase
Router# reload

! Часткове скидання (зі збереженням IOS)
Router# write erase
Router# delete vlan.dat
Router# reload
```

### 1.2 Процес завантаження та ROMMON

```
! Перервати завантаження: натисніть Ctrl+Break протягом перших 60 секунд

! Команди ROMMON:
rommon 1 > confreg 0x2142    ! обхід завантаження startup-config
rommon 2 > reset             ! перезавантаження
rommon 3 > dir flash:        ! список файлів у Flash
rommon 4 > boot flash:c2900-universalk9-mz.SPA.157-3.M.bin  ! завантажити конкретний IOS
rommon 5 > tftpdnld          ! завантаження IOS з TFTP-сервера
```

### 1.3 Процедура відновлення пароля

**Для маршрутизаторів (ISR G2/ASR):**

1. Підключитись консольним кабелем
2. Перезавантажте пристрій та увійдіть в ROMMON (`Ctrl+Break`)
3. `confreg 0x2142`
4. `reset`
5. Пропустити початкову конфігурацію
6. `enable`
7. `copy startup-config running-config`
8. Змініть паролі
9. `config-register 0x2102`
10. `copy running-config startup-config`
11. `reload`

**Для комутаторів (Catalyst):**

1. Вимкніть пристрій
2. Утримуйте кнопку Mode під час увімкнення комутатора
3. Відпустіть кнопку, коли світлові індикатори почнуть мигати
4. `flash_init`
5. `load_helper`
6. `rename flash:config.text flash:config.text.old`
7. `boot`
8. `enable`
9. `rename flash:config.text.old flash:config.text`
10. `copy flash:config.text system:running-config`
11. Змініть паролі
12. `copy running-config startup-config`

### 1.4 Апаратне забезпечення

```
! Переглянути відомості про "залізо"
Router# show inventory         ! перелік модулів та серійних номерів
Router# show diag              ! детальна діагностика
Router# show environment       ! температура, живлення, вентилятори
Router# show module            ! встановлені модулі
Router# show idprom backplane

! Cisco 2960X/3650/3850 — команди для стекованих шасі
Switch# show switch            ! список пристроїв у стеку
Switch# show switch detail     ! детально
Switch# show switch neighbors  ! сусіди по стеку
Switch# switch 1 provision ws-c3650-24ps
Switch# switch 1 priority 15  ! більше значення = головний пристрій
Switch# switch 2 renumber 3
Switch# switch 1 reload
```

### 1.5 Управління прошивкою IOS

```
! Перевірити поточну версію IOS
Router# show version           ! версія IOS, uptime, модель пристрою
Router# show boot
Router# show flash:
Router# dir flash:

! Оновити IOS через TFTP
Router# copy tftp://192.168.1.10/c2900-universalk9-mz.SPA.157-3.M.bin flash:
Router# boot system flash:c2900-universalk9-mz.SPA.157-3.M.bin
Router# copy running-config startup-config
Router# reload

! Встановлення пакетів Bundle/Install (Cat3K/4K)
Switch# archive download-sw /force-reload /overwrite tftp://192.168.1.10/cat3k_caa-universalk9.16.12.04.SPA.bin

! Режим встановлення (IOS XE)
Router# request platform software package install switch all file tftp://192.168.1.10/asr1000-universalk9.17.03.03.SPA.pkg
```

---

## 2 Навігація та основи IOS

### 2.1 Основи командного рядка

```
! Скорочення команд (до унікальних символів)
Router# conf t          ! configure terminal
Router# sh run          ! show running-config
Router# sh ip int br    ! show ip interface brief
Router# wr              ! write memory (copy run start)

! Комбінації клавіш для редагування
Ctrl+A          ! перейти на початок рядка
Ctrl+E          ! перейти в кінець рядка
Ctrl+W          ! видалити попереднє слово
Esc+B           ! перейти на слово назад
Esc+F           ! перейти на слово вперед
Ctrl+R          ! повторно відобразити рядок
Ctrl+U          ! видалити весь рядок
Ctrl+Y          ! вставити видалений текст
Tab             ! автодоповнення

! Фільтрація виводу команд
Router# show running-config | include ospf   ! рядки що містять "ospf"
Router# show running-config | section router ! секція "router"
Router# show running-config | exclude !      ! без рядків-коментарів
Router# show running-config | begin interface! починаючи з рядка "interface"
Router# show running-config | count ospf     ! кількість входжень
```

### 3.2 Контекстна довідка

```
Router# ?                           ! список доступних команд
Router# show ?                      ! параметри команди show
Router# configure ?                 ! параметри configure
Router# interface gigabitEthernet 0/0/0 ?
Router# show ip route <Tab>         ! автодоповнення

! Часткове доповнення
Router# sh ip ro<Tab>               ! доповнює до "show ip route"
```

### 3.3 Рівні привілеїв користувачів

```
! Налаштування рівнів привілеїв (0–15, де 15 = повний доступ)
Router(config)# privilege exec level 5 configure terminal
Router(config)# privilege exec level 5 show running-config
Router(config)# privilege interface level 5 shutdown
Router(config)# username admin privilege 5 secret P@ssw0rd

! Перегляд поточного рівня
Router# show privilege
Router# show privilege all

! Встановити пароль для рівня та увійти
Router(config)# enable secret level 5 SuperSecret123
Router# enable 5
```

### 3.4 Псевдоніми команд

```
! Створення власних псевдонімів команд
Router(config)# alias exec srs show running-config | section
Router(config)# alias exec shbr show ip interface brief
Router(config)# alias exec routes show ip route
Router(config)# alias exec neighbors show cdp neighbors detail
Router(config)# alias configure config t

! Постійний псевдонім для резервного копіювання
Router(config)# alias exec backup copy running-config tftp://192.168.1.10/
Router# backup R1-config.txt
```

### 3.5 Скрипти за допомогою TCL

```
! Вхід в TCL-оболонку для автоматизації команд
Router# tclsh
Router(tcl)# puts [exec "show version"]
Router(tcl)# foreach i {1 2 3 4 5} { puts "Interface Gi0/$i" }
Router(tcl)# exit

! Одноразова TCL-команда
Router# tclsh puts [exec "show ip interface brief"]
```

---

## 4 Робота з файловою системою

### 4.1 Навігація файловою системою

```
! Навігація по файловій системі
Router# pwd                     ! поточна директорія
Router# dir                     ! список файлів
Router# dir flash:              ! вміст Flash-пам'яті
Router# dir nvram:              ! вміст NVRAM
Router# dir usbflash0:          ! USB (якщо підключено)
Router# dir sdflash:            ! SD-карта (ASR/ISR)
Router# dir bootflash:          ! завантажувальна Flash
Router# dir system:             ! системні файли

! Операції з файлами
Router# copy source-url destination-url        ! копіювати файл
Router# copy running-config flash:backup-config! зберегти конфіг у Flash
Router# copy flash:config.text tftp://192.168.1.10/ ! копіювати на TFTP
Router# delete flash:old-config.text           ! видалити файл
Router# erase flash:                           ! ⚠️ очистити всю Flash
Router# mkdir flash:backups                    ! створити директорію
Router# rmdir flash:backups                    ! видалити директорію
```

### 4.2 Управління конфігурацією

```
! Архівування конфігурацій
Router# archive
Router(config-archive)# path flash:backups/$h-config  ! $h = hostname
Router(config-archive)# maximum 10            ! зберігати до 10 версій
Router(config-archive)# time-period 1440      ! авто-збереження кожні 24 год
Router(config-archive)# write-memory          ! архівувати при wr

! Перегляд та відновлення з архіву
Router# show archive
Router# configure replace flash:backups/R1-config-1  ! відкат до версії

! Відкат конфігурації
Router# configure replace flash:backup-config 10
Router# configure revert now
Router# configure confirm

! Точки відновлення (IOS XE)
Router# checkpoint database
Router# show checkpoint database
Router# rollback checkpoint checkpoint_name
```

### 4.3 Робота з SCP

```
! Налаштувати SCP-сервер на пристрої
Router(config)# ip scp server enable
Router(config)# username admin secret admin123
Router(config)# ip ssh version 2

! Копіювання через SCP (зашифровано, на відміну від TFTP)
Router# copy running-config scp://admin@192.168.1.10/backups/R1-config
Router# copy scp://admin@192.168.1.10/configs/new-config running-config

! З Linux на маршрутизатор
# scp R1-config admin@10.0.0.1:flash:
```

### 4.4 Робота з USB

```
! Перевірити стан USB
Router# show usb
Router# dir usbflash0:

! Копіювати на/з USB
Router# copy running-config usbflash0:/backup-config
Router# copy usbflash0:/new-ios.bin flash:

! Завантажитись з USB
Router(config)# boot system usbflash0:c2900-universalk9-mz.SPA.157-3.M.bin
```

### 4.5 Автоматизація бекапів

```
! EEM-скрипт для автоматичного резервного копіювання щодня о 02:00
Router(config)# event manager applet AUTO-BACKUP
Router(config-applet)# event timer cron cron-entry "0 2 * * *"
Router(config-applet)# action 1.0 cli command "enable"
Router(config-applet)# action 2.0 cli command "copy running-config tftp://192.168.1.10/backups/$_device_name-$_event_pub_time"
Router(config-applet)# action 3.0 syslog msg "Backup completed successfully"
```

---

## 5 Управління ліцензіями

### 5.1 Типи та моделі ліцензій

```
! Переглянути інформацію про ліцензії
Router# show license
Router# show license feature
Router# show license udi                       ! унікальний ідентифікатор пристрою
Router# show license right-to-use
Router# show license tech-support

! Smart Licensing (IOS XE)
Router# show license status
Router# show license authorization
Router# license smart register idtoken XXXX-XXXX-XXXX-XXXX
Router# license smart reservation request local
Router# license smart reservation install file flash:reservation.lic

! Traditional Licensing (класичний варіант)
Router# show license all
Router# license install flash:license.lic
Router# license boot level ipbasek9
Router# reload
```

### 5.2 Робота з ліцензіями

```
! Зберегти ліцензію у файл
Router# license save flash:all_licenses.lic

! Експорт/імпорт ліцензії
Router# license export flash:license_pa.lic
Router# license import flash:new_license.lic

! Ознайомча (Evaluation) ліцензія
Router# license accept end user agreement
Router# license boot level securityk9
Router# reload

! Перевірити відповідність ліцензій
Router# show license violation
```

### 5.3 Right-to-Use (RTU) Ліцензування

```
! Управління RTU-ліцензіями
Router# license right-to-use activate ipservices
Router# license right-to-use deactivate ipservices
Router# show license right-to-use
Router# license right-to-use perpetual ipservices accept
```

---

## 6 Управління пристроєм

### 6.1 Системні налаштування

```
! Базове налаштування пристрою
Router(config)# hostname R1-CORE
Router(config)# no ip domain-lookup            ! вимкнути DNS-запити в CLI
Router(config)# ip domain-name company.com
Router(config)# service password-encryption    ! шифрувати паролі у конфізі
Router(config)# service timestamps debug datetime msec localtime show-timezone
Router(config)# service timestamps log datetime msec localtime show-timezone
Router(config)# logging buffered 16384 debugging
Router(config)# logging console critical
Router(config)# logging monitor debugging
Router(config)# logging host 192.168.100.10
Router(config)# logging source-interface Loopback0

! Налаштування банерів
Router(config)# banner motd ^
************************************************************
*              AUTHORIZED ACCESS ONLY                      *
*  This system is the property of Company Inc.             *
*  Unauthorized access is prohibited and will be prosecuted. *
************************************************************
^

Router(config)# banner login ^
Please enter your credentials to access this device.
Contact Network Operations for assistance.
^

Router(config)# banner exec ^
Welcome to R1-CORE. You are logged in as $username.
Current time: $_timenow
^
```

### 6.2 Налаштування NTP

```
! Сервери NTP для синхронізації часу
Router(config)# ntp server 0.pool.ntp.org prefer
Router(config)# ntp server 1.pool.ntp.org
Router(config)# ntp server 2.pool.ntp.org
Router(config)# ntp server 192.168.100.10

! Автентифікація NTP
Router(config)# ntp authenticate
Router(config)# ntp authentication-key 1 md5 NTP-KEY-123
Router(config)# ntp trusted-key 1
Router(config)# ntp server 0.pool.ntp.org key 1

! NTP Master/Peer
Router(config)# ntp master 3               ! зробити пристрій NTP-сервером (stratum 3)
Router(config)# ntp peer 192.168.1.2
Router(config)# ntp update-calendar

! Контроль доступу до NTP
Router(config)# ntp access-group peer 10
Router(config)# ntp access-group serve-only 20
Router(config)# ntp access-group serve 30
Router(config)# ntp access-group query-only 40

! Перевірка
Router# show ntp associations
Router# show ntp status
Router# show ntp clock
Router# show clock detail
```

### 6.3 Налаштування DNS

```
! DNS-сервери
Router(config)# ip name-server 8.8.8.8
Router(config)# ip name-server 8.8.4.4
Router(config)# ip name-server 192.168.100.10
Router(config)# ip domain-name company.com
Router(config)# ip domain-list branch.company.com
Router(config)# ip domain-list partner.com
Router(config)# ip domain lookup source-interface Loopback0

! Статичні DNS-записи (hosts)
Router(config)# ip host router1.company.com 192.168.1.1
Router(config)# ip host switch1 10.0.0.1 10.0.0.2
Router(config)# ip host ftp-server 192.168.100.10

! DNS Caching
Router(config)# ip dns server
Router(config)# ip dns cache
Router(config)# ip dns spoofing 192.168.1.1

! Перевірка
Router# show hosts
Router# nslookup router1.company.com
Router# ping router1.company.com
```

### 6.4 Налаштування Syslog

```
! Налаштування Syslog-сервера
Router(config)# logging on
Router(config)# logging host 192.168.100.10
Router(config)# logging host 192.168.100.11
Router(config)# logging trap debugging          ! рівень: debug/info/warning/error/critical
Router(config)# logging facility local7
Router(config)# logging source-interface Loopback0
Router(config)# logging origin-id hostname
Router(config)# logging sequence-numbers
Router(config)# logging discriminator OSPF msg-body includes "OSPF"

! Локальне логування (в буфер пристрою)
Router(config)# logging buffered 16384 informational
Router(config)# logging console warnings
Router(config)# logging monitor debugging
Router(config)# logging persistent url flash:logs size 4096
Router(config)# logging history debugging

! Фільтри логів
Router(config)# logging filter OSPF ROUTER-ID 1.1.1.1
Router(config)# logging filter BGP neighbor 192.168.1.2

! Перевірка
Router# show logging
Router# show logging history
Router# show logging filter
```

### 6.5 Налаштування SNMP

```
! SNMPv2c (простий, без шифрування)
Router(config)# snmp-server community RO-COMMUNITY RO 10
Router(config)# snmp-server community RW-COMMUNITY RW 20
Router(config)# snmp-server host 192.168.100.10 version 2c RO-COMMUNITY
Router(config)# snmp-server enable traps
Router(config)# snmp-server trap-source Loopback0
Router(config)# snmp-server queue-length 100
Router(config)# snmp-server location "Data Center Rack A1"
Router(config)# snmp-server contact "Network Operations noc@company.com"
Router(config)# snmp-server chassis-id R1-CORE-ASR1001

! SNMPv3 (безпечний варіант з шифруванням)
Router(config)# snmp-server group ADMIN v3 priv read ALL write ALL
Router(config)# snmp-server user admin1 ADMIN v3 auth sha AuthPass123 priv aes 128 PrivPass123
Router(config)# snmp-server host 192.168.100.10 version 3 priv admin1
Router(config)# snmp-server enable traps snmp authentication linkdown coldstart warmstart
Router(config)# snmp-server enable traps config
Router(config)# snmp-server enable traps entity
Router(config)# snmp-server enable traps cpu threshold
Router(config)# snmp-server enable traps memory threshold
Router(config)# snmp-server enable traps ospf
Router(config)# snmp-server enable traps bgp

! SNMP Views (обмеження видимих OID)
Router(config)# snmp-server view ALL iso included
Router(config)# snmp-server view RESTRICTED system included
Router(config)# snmp-server view RESTRICTED interfaces excluded

! Перевірка
Router# show snmp
Router# show snmp user
Router# show snmp group
Router# show snmp host
```

---

## 7 Основи безпеки

### 7.1 Управління паролями

```
! Налаштування паролів
Router(config)# enable secret SuperSecret123!
Router(config)# enable algorithm-type scrypt secret EvenMoreSecret456!  ! сильніше хешування
Router(config)# username admin privilege 15 secret P@ssw0rd123!
Router(config)# username operator privilege 5 secret 0p3rator!
Router(config)# service password-encryption      ! шифрувати всі паролі у конфізі
Router(config)# security passwords min-length 10 ! мінімальна довжина пароля
Router(config)# login block-for 300 attempts 3 within 60  ! блокувати після 3 невдалих спроб за 60 сек
Router(config)# login delay 2                    ! затримка між спробами входу
Router(config)# login on-failure log
Router(config)# login on-success log

! Налаштування ліній (console/vty)
Router(config-line)# password Cons0leP@ss
Router(config-line)# login local
Router(config-line)# exec-timeout 5 0            ! автовихід через 5 хв бездіяльності
Router(config-line)# absolute-timeout 30          ! примусовий вихід через 30 хв
Router(config-line)# session-timeout 30
Router(config-line)# logout-warning 5

! Заборона відновлення пароля
Router(config)# no service password-recovery
Router(config)# enable secret recovery SecretRecovery789!
```

### 7.2 Налаштування SSH

```
! Налаштування SSH v2
Router(config)# ip ssh version 2
Router(config)# ip ssh time-out 60
Router(config)# ip ssh authentication-retries 2
Router(config)# ip ssh maxstartups 3
Router(config)# ip ssh logging events
Router(config)# ip ssh server algorithm mac hmac-sha2-256 hmac-sha2-512
Router(config)# ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
Router(config)# ip ssh server algorithm hostkey rsa-sha2-256 rsa-sha2-512
Router(config)# ip ssh server disable-deprecated-version 1.99
Router(config)# ip ssh server max-sessions-per-connection 10

! Автентифікація за ключем (замість пароля)
Router(config)# crypto key generate rsa modulus 4096
Router(config)# crypto key generate dsa modulus 2048
Router(config)# crypto key generate ecdsa curve 256
Router(config)# ip ssh pubkey-chain
Router(config-ssh-pubkey)# username admin
Router(config-ssh-pubkey)# key-hash ssh-rsa AAAA...==
Router(config-ssh-pubkey)# exit

! Source Interface для SSH
Router(config)# ip ssh source-interface Loopback0
Router(config)# ip ssh vrf MGMT

! Перевірка
Router# show ip ssh
Router# show ssh
Router# show crypto key mypubkey rsa
```

### 7.3 Сервіси TCP/UDP

```
! Вимкнути непотрібні сервіси (hardening)
Router(config)# no service tcp-small-servers
Router(config)# no service udp-small-servers
Router(config)# no service finger
Router(config)# no ip bootp server
Router(config)# no ip http server
Router(config)# no ip http secure-server
Router(config)# no ip source-route
Router(config)# no ip gratuitous-arps
Router(config)# no cdp run                  ! вимкнути CDP глобально
Router(config)# no lldp run                 ! вимкнути LLDP глобально
Router(config)# no ip proxy-arp
Router(config)# no ip unreachables
Router(config)# no ip redirects
Router(config)# no ip mask-reply

! Вимкнути на конкретному інтерфейсі
Router(config-if)# no cdp enable
Router(config-if)# no lldp transmit
Router(config-if)# no lldp receive
Router(config-if)# no ip directed-broadcast
Router(config-if)# no ip unreachables
Router(config-if)# no ip proxy-arp
```

### 7.4 Політики Control Plane (CoPP)

```
! CoPP — обмежує трафік до CPU маршрутизатора
Router(config)# class-map match-any COPP-CRITICAL
Router(config-cmap)# match access-group 110
Router(config-cmap)# exit

Router(config)# class-map match-any COPP-IMPORTANT
Router(config-cmap)# match access-group 120
Router(config-cmap)# exit

Router(config)# policy-map COPP-POLICY
Router(config-pmap)# class COPP-CRITICAL
Router(config-pmap-c)# police 8000 conform-action transmit exceed-action drop
Router(config-pmap-c)# exit
Router(config-pmap)# class COPP-IMPORTANT
Router(config-pmap-c)# police 40000 conform-action transmit exceed-action drop
Router(config-pmap-c)# exit
Router(config-pmap)# class class-default
Router(config-pmap-c)# police 10000 conform-action transmit exceed-action drop
Router(config-pmap-c)# exit
Router(config-pmap)# exit

Router(config)# control-plane
Router(config-cp)# service-policy input COPP-POLICY
Router(config-cp)# exit

! ACL для CoPP
Router(config)# access-list 110 permit tcp any any eq 22
Router(config)# access-list 110 permit tcp any any eq 23
Router(config)# access-list 120 permit ospf any any
Router(config)# access-list 120 permit eigrp any any
Router(config)# access-list 120 permit pim any any
```

---

## 8 AAA & Управління доступом

### 8.1 Основи AAA

```
! Увімкнути AAA (Authentication, Authorization, Accounting)
Router(config)# aaa new-model
Router(config)# aaa authentication login default local
Router(config)# aaa authentication enable default enable
Router(config)# aaa authorization exec default local
Router(config)# aaa accounting exec default start-stop group radius

! Локальна база користувачів
Router(config)# username admin privilege 15 algorithm-type scrypt secret AdminPass123!
Router(config)# username operator privilege 5 secret 0p3rator!
Router(config)# username guest privilege 1 secret GuestAccess!

! Паролі для рівнів enable
Router(config)# enable secret level 15 SuperSecret!
Router(config)# enable secret level 5 OperatorPass
```

### 8.2 Налаштування RADIUS

```
! Налаштування RADIUS-серверів
Router(config)# radius server ISE-PRIMARY
Router(config-radius-server)# address ipv4 192.168.100.10 auth-port 1812 acct-port 1813
Router(config-radius-server)# key RadiusKey123!
Router(config-radius-server)# retransmit 3
Router(config-radius-server)# timeout 5
Router(config-radius-server)# exit

Router(config)# radius server ISE-SECONDARY
Router(config-radius-server)# address ipv4 192.168.100.11 auth-port 1812 acct-port 1813
Router(config-radius-server)# key RadiusKey456!
Router(config-radius-server)# exit

! Група серверів RADIUS
Router(config)# aaa group server radius ISE-GROUP
Router(config-sg-radius)# server name ISE-PRIMARY
Router(config-sg-radius)# server name ISE-SECONDARY
Router(config-sg-radius)# exit

! AAA з RADIUS
Router(config)# aaa authentication login default group ISE-GROUP local
Router(config)# aaa authentication enable default group ISE-GROUP enable
Router(config)# aaa authorization exec default group ISE-GROUP local
Router(config)# aaa accounting exec default start-stop group ISE-GROUP
Router(config)# aaa accounting network default start-stop group ISE-GROUP
```

### 8.3 Налаштування TACACS+

```
! Налаштування TACACS+-серверів
Router(config)# tacacs server TACACS-PRIMARY
Router(config-server-tacacs)# address ipv4 192.168.100.20
Router(config-server-tacacs)# key TacacsKey123!
Router(config-server-tacacs)# single-connection
Router(config-server-tacacs)# timeout 10
Router(config-server-tacacs)# exit

! Група серверів TACACS+
Router(config)# aaa group server tacacs+ TACACS-GROUP
Router(config-sg-tacacs)# server name TACACS-PRIMARY
Router(config-sg-tacacs)# exit

! AAA з TACACS+
Router(config)# aaa authentication login default group TACACS-GROUP local
Router(config)# aaa authentication enable default group TACACS-GROUP enable
Router(config)# aaa authorization commands 15 default group TACACS-GROUP local
Router(config)# aaa authorization config-commands
Router(config)# aaa authorization exec default group TACACS-GROUP local
Router(config)# aaa accounting commands 15 default start-stop group TACACS-GROUP
Router(config)# aaa accounting exec default start-stop group TACACS-GROUP
```

### 8.4 Авторизація команд

```
! Рівні авторизації команд
Router(config)# privilege exec level 5 show running-config
Router(config)# privilege exec level 5 show interfaces
Router(config)# privilege exec level 10 configure terminal
Router(config)# privilege exec level 15 reload
Router(config)# privilege interface level 5 shutdown
Router(config)# privilege interface level 10 ip address

! CLI Views — набори дозволених команд для різних ролей
Router(config)# parser view ROOT
Router(config-view)# secret RootView123!
Router(config-view)# commands exec include all show
Router(config-view)# commands exec include configure
Router(config-view)# commands exec include reload
Router(config-view)# exit

Router(config)# parser view OPERATOR
Router(config-view)# secret OperatorView456!
Router(config-view)# commands exec include show
Router(config-view)# commands exec exclude show running-config  ! заборонити конкретну команду
Router(config-view)# commands exec include ping
Router(config-view)# commands exec include traceroute
Router(config-view)# exit
```

### 8.5 Downloadable ACLs

```
! ACL, що завантажується з RADIUS після автентифікації
Router(config)# ip access-list extended DYNACL-WEBSERVER
Router(config-ext-nacl)# permit tcp any host 192.168.1.10 eq 80
Router(config-ext-nacl)# permit tcp any host 192.168.1.10 eq 443
Router(config-ext-nacl)# deny ip any any
Router(config-ext-nacl)# exit

! AAA атрибут для ACL
Router(config)# aaa authorization network default group ISE-GROUP
```

---

## 9 Протоколи управління

### 9.1 Управління HTTP/HTTPS

```
! Налаштування HTTP/HTTPS для веб-інтерфейсу
Router(config)# ip http server
Router(config)# ip http port 8080
Router(config)# ip http authentication local
Router(config)# ip http access-class 10            ! обмежити доступ ACL
Router(config)# ip http secure-server
Router(config)# ip http secure-port 443
Router(config)# ip http secure-ciphersuite aes256-cbc-sha1
Router(config)# ip http timeout-policy idle 60 life 86400 requests 100

! ACL для обмеження доступу до HTTP
Router(config)# access-list 10 permit 192.168.100.0 0.0.0.255
Router(config)# access-list 10 deny any

! Перевірка
Router# show ip http server status
Router# show ip http secure-server status
```

### 9.2 Netconf/RESTCONF

```
! NETCONF — програмне управління пристроєм (через SSH, порт 830)
Router(config)# netconf-yang
Router(config)# netconf-yang feature candidate-datastore
Router(config)# netconf-yang feature notify
Router(config)# netconf-yang feature url
Router(config)# netconf-yang ssh port 830

! RESTCONF — HTTP-API для управління пристроєм
Router(config)# restconf
Router(config)# ip http secure-server
Router(config)# restconf data-entries 10000

! Перевірка
Router# show netconf-yang sessions
Router# show restconf
```

### 9.3 gRPC/Telemetry

```
! gRPC — протокол для streaming-телеметрії
Router(config)# grpc
Router(config-grpc)# port 57400
Router(config-grpc)# no-tls
Router(config-grpc)# address-family ipv4
Router(config-grpc)# max-request-total 64
Router(config-grpc)# max-request-per-user 16

! Телеметрія — потокова передача метрик на колектор
Router(config)# telemetry ietf subscription 101
Router(config-telemetry)# encoding encode-kvgpb
Router(config-telemetry)# filter xpath /interfaces/interface/statistics
Router(config-telemetry)# source-address 192.168.1.1
Router(config-telemetry)# stream yang-push
Router(config-telemetry)# update-policy periodic 500
Router(config-telemetry)# receiver ip address 192.168.100.30 5432 protocol grpc-tcp
```

### 9.4 NETCONF/YANG моделі

```
! Перевірка YANG-моделей
Router# show platform software yang-management process
Router# show yang-operational memory
Router# show yang schema

! Встановлення YANG-моделей
Router# copy tftp://192.168.1.10/ietf-interfaces@2014-05-08.yang flash:
Router# install add file flash:ietf-interfaces@2014-05-08.yang activate

! NETCONF-операції (з Linux)
# netconf-console --host 10.0.0.1 --port 830 --user admin --password pass --get-config
```

---

## 10 Комутація Ethernet

### 10.1 Таблиця MAC-адрес

```
! Переглянути таблицю MAC-адрес
Switch# show mac address-table
Switch# show mac address-table dynamic
Switch# show mac address-table aging-time
Switch# show mac address-table count
Switch# show mac address-table interface gigabitEthernet 1/0/1
Switch# show mac address-table vlan 10
Switch# show mac address-table address 0050.7966.6800

! Налаштування таблиці MAC
Switch(config)# mac address-table aging-time 300           ! час старіння запису (сек)
Switch(config)# mac address-table learning vlan 10         ! дозволити навчання
Switch(config)# no mac address-table learning vlan 20      ! заборонити навчання
Switch(config)# mac address-table static 0050.7966.6800 vlan 10 interface gi1/0/1  ! статичний запис
Switch(config)# mac address-table notification change
Switch(config)# mac address-table limit maximum 1000 vlan 10

! Очистити таблицю MAC
Switch# clear mac address-table dynamic
Switch# clear mac address-table dynamic interface gi1/0/1
Switch# clear mac address-table dynamic vlan 10
Switch# clear mac address-table dynamic address 0050.7966.6800
```

### 10.2 Налаштування портів

```
! Базове налаштування порту
Switch(config)# interface gigabitEthernet 1/0/1
Switch(config-if)# description "Connected to Server01"
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# switchport voice vlan 20
Switch(config-if)# spanning-tree portfast           ! прискорити вхід у forwarding
Switch(config-if)# spanning-tree bpduguard enable   ! захист від несанкціонованих SW
Switch(config-if)# no shutdown
Switch(config-if)# exit

! Швидкість та дуплекс
Switch(config-if)# speed 1000
Switch(config-if)# duplex full
Switch(config-if)# negotiation auto
Switch(config-if)# mtu 9216                         ! jumbo frames

! Відновлення портів з err-disabled стану
Switch(config)# errdisable recovery cause psecure-violation
Switch(config)# errdisable recovery cause bpduguard
Switch(config)# errdisable recovery interval 30
Switch(config)# errdisable detect cause all

! Шаблони інтерфейсів (застосувати однакові налаштування до багатьох портів)
Switch(config)# template USER-PORT
Switch(config-template)# switchport mode access
Switch(config-template)# switchport access vlan 10
Switch(config-template)# spanning-tree portfast
Switch(config-template)# spanning-tree bpduguard enable
Switch(config-template)# storm-control broadcast level 10.00
Switch(config-template)# service-policy input AUTOQOS-VOIP
Switch(config-template)# exit

Switch(config)# interface range gi1/0/1 - 24
Switch(config-if-range)# source template USER-PORT
```

### 10.3 CDP/LLDP

```
! CDP — виявлення сусідніх Cisco-пристроїв
Switch(config)# cdp run
Switch(config)# cdp timer 60                        ! інтервал надсилання (сек)
Switch(config)# cdp holdtime 180                    ! час утримання інформації
Switch(config)# cdp advertise-v2
Switch(config-if)# cdp enable
Switch(config-if)# no cdp enable                    ! вимкнути на порту

! LLDP — стандартний протокол виявлення (будь-який виробник)
Switch(config)# lldp run
Switch(config)# lldp timer 30
Switch(config)# lldp holdtime 120
Switch(config)# lldp reinit 2
Switch(config-if)# lldp transmit
Switch(config-if)# lldp receive
Switch(config-if)# lldp med

! Перевірка
Switch# show cdp neighbors
Switch# show cdp neighbors detail
Switch# show cdp traffic
Switch# show lldp neighbors
Switch# show lldp neighbors detail
Switch# show lldp traffic
Switch# show lldp interface gi1/0/1
```

### 10.4 UDLD & Loop Guard

```
! UDLD — виявлення однонаправлених зв'язків
Switch(config)# udld enable
Switch(config)# udld aggressive              ! агресивний режим: блокує порт при збої
Switch(config-if)# udld port aggressive
Switch(config-if)# udld disable

! Loop Guard — захист від петель через втрату BPDU
Switch(config)# spanning-tree loopguard default
Switch(config-if)# spanning-tree guard loop
Switch(config-if)# spanning-tree guard root

! Перевірка
Switch# show udld neighbors
Switch# show udld gi1/0/1
Switch# show spanning-tree inconsistentports
```

---

## 11 VLANs & Trunking

### 11.1 Налаштування VLAN

```
! Створення VLAN
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# state active
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name ENGINEERING
Switch(config-vlan)# exit

Switch(config)# vlan 99
Switch(config-vlan)# name MANAGEMENT
Switch(config-vlan)# exit

! Діапазон VLAN
Switch(config)# vlan 100-200
Switch(config-vlan)# name USER-VLANS
Switch(config-vlan)# exit

! Extended VLANs (1006–4094)
Switch(config)# vlan 2000
Switch(config-vlan)# name EXTENDED-VLAN
Switch(config-vlan)# exit

! Private VLANs — ізоляція в межах VLAN
Switch(config)# vlan 500
Switch(config-vlan)# private-vlan primary
Switch(config-vlan)# private-vlan association 501-502
Switch(config-vlan)# exit

Switch(config)# vlan 501
Switch(config-vlan)# private-vlan isolated          ! порти не бачать одне одного
Switch(config-vlan)# exit

Switch(config)# vlan 502
Switch(config-vlan)# private-vlan community         ! порти бачать одне одного
Switch(config-vlan)# exit
```

### 11.2 VLAN Trunking

```
! Налаштування транк-порту (передає кілька VLAN)
Switch(config)# interface gigabitEthernet 1/0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport trunk native vlan 99       ! нетегований VLAN
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# switchport trunk allowed vlan add 40-50
Switch(config-if)# switchport trunk allowed vlan remove 30
Switch(config-if)# switchport trunk pruning vlan 2-1001
Switch(config-if)# switchport nonegotiate               ! вимкнути DTP
Switch(config-if)# spanning-tree guard root
Switch(config-if)# exit

! Dynamic Trunking Protocol (DTP)
Switch(config-if)# switchport mode dynamic desirable     ! активно ініціює транк
Switch(config-if)# switchport mode dynamic auto          ! чекає ініціативи

! VLAN Translation (перетворення номерів VLAN)
Switch(config-if)# switchport vlan mapping 10 110
Switch(config-if)# switchport vlan mapping 20 120

! Перевірка
Switch# show interfaces trunk
Switch# show interfaces gi1/0/24 switchport
Switch# show interfaces gi1/0/24 trunk
Switch# show vlan
Switch# show vlan id 10
Switch# show vlan name SALES
```

### 11.3 Налаштування VTP

```
! VTP — синхронізація VLAN між комутаторами
Switch(config)# vtp domain COMPANY
Switch(config)# vtp mode server      ! server = створює/змінює VLAN
Switch(config)# vtp password VTP-Pass123!
Switch(config)# vtp version 2
Switch(config)# vtp pruning          ! не надсилати трафік VLAN туди, де їх немає
Switch(config)# vtp interface Loopback0

! VTP Client (отримує VLAN від server, не може створювати)
Switch(config)# vtp mode client
Switch(config)# vtp transparent      ! не бере участі в VTP, але пропускає його

! Перевірка
Switch# show vtp status
Switch# show vtp password
Switch# show vtp counters
Switch# show vlan brief
```

### 11.4 Voice VLAN

```
! Налаштування Voice VLAN для IP-телефонів
Switch(config)# interface gigabitEthernet 1/0/10
Switch(config-if)# switchport voice vlan 110
Switch(config-if)# switchport priority extend cos 0
Switch(config-if)# auto qos voip trust
Switch(config-if)# mls qos trust cos              ! довіряти CoS-мітці від телефону
Switch(config-if)# spanning-tree portfast
Switch(config-if)# spanning-tree bpduguard enable

! LLDP-MED для IP-телефонів
Switch(config)# lldp run
Switch(config-if)# lldp med
Switch(config-if)# lldp med-tlv-select inventory-management network-policy
```

### 11.5 VLAN ACLs (VACLs)

```
! VACL — фільтрація трафіку всередині VLAN
Switch(config)# vlan access-map BLOCK-SERVER 10
Switch(config-access-map)# match ip address SERVER-ACL
Switch(config-access-map)# action drop
Switch(config-access-map)# exit

Switch(config)# vlan access-map BLOCK-SERVER 20
Switch(config-access-map)# action forward
Switch(config-access-map)# exit

Switch(config)# vlan filter BLOCK-SERVER vlan-list 10  ! застосувати до VLAN 10

! ACL для VACL
Switch(config)# ip access-list extended SERVER-ACL
Switch(config-ext-nacl)# deny tcp any host 192.168.10.10 eq 3389
Switch(config-ext-nacl)# permit ip any any
```

---

## 12 Spanning Tree Protocol

### 12.1 Основи налаштування STP

```
! Вибір режиму STP
Switch(config)# spanning-tree mode rapid-pvst    ! Rapid PVST+ — рекомендовано
Switch(config)# spanning-tree mode mst           ! MST — для великих мереж
Switch(config)# spanning-tree mode pvst          ! класичний PVST

! Пріоритет мосту (менше = більше шансів стати Root Bridge)
Switch(config)# spanning-tree vlan 1,10,20,30 priority 4096
Switch(config)# spanning-tree vlan 40-50 priority 8192

! Захист Root Bridge
Switch(config)# spanning-tree portfast bpduguard default
Switch(config-if)# spanning-tree guard root
Switch(config-if)# spanning-tree bpdufilter enable
Switch(config-if)# spanning-tree bpduguard enable

! Перевірка
Switch# show spanning-tree
Switch# show spanning-tree vlan 10
Switch# show spanning-tree interface gi1/0/1
Switch# show spanning-tree detail
Switch# show spanning-tree inconsistentports
Switch# show spanning-tree mst configuration
```

### 12.2 Налаштування MST

```
! Налаштування MST-регіону
Switch(config)# spanning-tree mst configuration
Switch(config-mst)# name REGION1
Switch(config-mst)# revision 1
Switch(config-mst)# instance 1 vlan 10,20,30    ! прив'язати VLAN до instance
Switch(config-mst)# instance 2 vlan 40,50,60
Switch(config-mst)# exit

! Параметри MST-instance
Switch(config)# spanning-tree mst 1 priority 4096
Switch(config)# spanning-tree mst 2 priority 8192
Switch(config)# spanning-tree mst 0-2 hello-time 2
Switch(config)# spanning-tree mst 0-2 forward-time 15
Switch(config)# spanning-tree mst 0-2 max-age 20

! Перевірка
Switch# show spanning-tree mst
Switch# show spanning-tree mst configuration
Switch# show spanning-tree mst 1
Switch# show spanning-tree mst interface gi1/0/1
```

### 12.3 Таймери та оптимізація STP

```
! Таймери STP
Switch(config)# spanning-tree vlan 1 hello-time 2      ! BPDU кожні 2 сек
Switch(config)# spanning-tree vlan 1 forward-time 15   ! затримка переходу
Switch(config)# spanning-tree vlan 1 max-age 20        ! час до визнання Root недоступним
Switch(config)# spanning-tree transmit hold-count 6

! PortFast & BPDU Guard
Switch(config)# spanning-tree portfast default         ! увімкнути глобально для access-портів
Switch(config)# spanning-tree portfast bpduguard default
Switch(config-if)# spanning-tree portfast
Switch(config-if)# spanning-tree portfast trunk

! UplinkFast & BackboneFast (тільки для PVST)
Switch(config)# spanning-tree uplinkfast               ! швидший переключення uplink
Switch(config)# spanning-tree backbonefast             ! швидше виявлення змін топології

! Loop Guard
Switch(config)# spanning-tree loopguard default
Switch(config-if)# spanning-tree guard loop
```

### 12.4 Безпека STP

```
! BPDU Guard — вимкнути порт якщо прийшов BPDU (захист від зайвих SW)
Switch(config)# spanning-tree portfast bpduguard default
Switch(config)# errdisable recovery cause bpduguard
Switch(config)# errdisable recovery interval 30

! BPDU Filter — не надсилати та не обробляти BPDU на порту
Switch(config)# spanning-tree portfast bpdufilter default
Switch(config-if)# spanning-tree bpdufilter enable

! Root Guard — заборонити порту ставати root port
Switch(config-if)# spanning-tree guard root

! TC Guard — обмежити обробку Topology Change повідомлень
Switch(config)# spanning-tree tc-guard
Switch(config)# spanning-tree tc-filter

! Перевірка
Switch# show spanning-tree summary
Switch# show spanning-tree detail
Switch# debug spanning-tree events
```

---

## 13 EtherChannel & LACP

### 13.1 Налаштування EtherChannel

```
! LACP Configuration (рекомендовано)
Switch(config)# interface range gigabitEthernet 1/0/1-2
Switch(config-if-range)# channel-group 1 mode active    ! активно надсилає LACP PDU
Switch(config-if-range)# channel-protocol lacp
Switch(config-if-range)# lacp port-priority 1000        ! менше = вищий пріоритет
Switch(config-if-range)# lacp rate fast
Switch(config-if-range)# exit

! PAgP Configuration (протокол Cisco)
Switch(config-if-range)# channel-group 1 mode desirable
Switch(config-if-range)# channel-protocol pagp
Switch(config-if-range)# pagp port-priority 1000

! Static EtherChannel (без протоколу, обидва боки = on)
Switch(config-if-range)# channel-group 1 mode on

! Port-Channel Interface — логічний інтерфейс для групи
Switch(config)# interface port-channel 1
Switch(config-if)# description "Uplink to Core"
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# lacp system-priority 1000
Switch(config-if)# lacp system-id 0050.7966.6800
Switch(config-if)# exit
```

### 13.2 Балансування навантаження

```
! Алгоритми балансування трафіку по каналах
Switch(config)# port-channel load-balance src-dst-ip    ! за IP-адресами
Switch(config)# port-channel load-balance src-dst-mac   ! за MAC-адресами
Switch(config)# port-channel load-balance src-dst-port  ! за TCP/UDP портами
Switch(config)# port-channel load-balance src-ip
Switch(config)# port-channel load-balance dst-ip
Switch(config)# port-channel load-balance src-mac
Switch(config)# port-channel load-balance dst-mac

! Балансування на рівні VLAN
Switch(config)# vlan 10
Switch(config-vlan)# lacp load-balancing src-dst-ip
Switch(config-vlan)# exit

! Перевірка
Switch# show etherchannel summary
Switch# show etherchannel port-channel
Switch# show etherchannel load-balance
Switch# show lacp neighbor
Switch# show lacp internal
Switch# show lacp counters
Switch# show pagp neighbor
Switch# show pagp internal
```

### 13.3 Розширене налаштування EtherChannel

```
! Cross-Stack EtherChannel (StackWise)
Switch(config)# interface port-channel 10
Switch(config-if)# stack-port 1/1-1/2
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

! VSS EtherChannel
Switch(config)# interface port-channel 100
Switch(config-if)# vss-port-channel
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

! Мінімальна кількість активних портів
Switch(config-if)# lacp min-links 2
Switch(config)# port-channel min-links 2

! Максимальна кількість активних портів у bundle
Switch(config-if)# lacp max-bundle 8

! Graceful Shutdown
Switch(config-if)# lacp graceful-convergence
```

### 13.4 Пошук несправностей в EtherChannel

```
! Діагностика проблем
Switch# show etherchannel detail
Switch# show etherchannel inconsistency  ! виявити розбіжності в налаштуваннях
Switch# debug etherchannel
Switch# debug lacp
Switch# debug pagp

! Скинути лічильники
Switch# clear lacp counters
Switch# clear pagp counters
Switch(config)# default interface port-channel 1  ! скинути налаштування port-channel

! Перенастройка порту вручну
Switch(config)# interface gi1/0/1
Switch(config-if)# shutdown
Switch(config-if)# no channel-group
Switch(config-if)# no shutdown
Switch(config-if)# channel-group 1 mode active
```

---

## 14 IP адресація

### 14.1 Налаштування IP-адрес

```
! Базова IP-адресація
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# ip address 10.0.0.1 255.255.255.0 secondary  ! другорядна адреса
Router(config-if)# ip directed-broadcast
Router(config-if)# ip unreachables
Router(config-if)# ip redirects
Router(config-if)# ip proxy-arp
Router(config-if)# no shutdown

! Loopback-інтерфейс (завжди активний, використовується як Router ID)
Router(config)# interface loopback 0
Router(config-if)# ip address 1.1.1.1 255.255.255.255
Router(config-if)# description "Router ID and Management"
Router(config-if)# ip ospf network point-to-point

! VLSM — маски змінної довжини
Router(config-if)# ip address 172.16.1.1 255.255.255.192   ! /26
Router(config-if)# ip address 10.1.1.1 255.255.255.224     ! /27
Router(config-if)# ip address 192.168.1.129 255.255.255.240! /28

! IPv6
Router(config-if)# ipv6 enable
Router(config-if)# ipv6 address 2001:db8::1/64
Router(config-if)# ipv6 address fe80::1 link-local
Router(config-if)# ipv6 address autoconfig
```

### 14.2 Сервіси DHCP

```
! DHCP-сервер
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10  ! резерв для статики
Router(config)# ip dhcp excluded-address 192.168.1.254
Router(config)# ip dhcp pool LAN-POOL
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
Router(dhcp-config)# domain-name company.com
Router(dhcp-config)# lease 8                            ! оренда 8 днів
Router(dhcp-config)# option 150 ip 192.168.1.10         ! TFTP для VoIP
Router(dhcp-config)# client-identifier 0100.5056.c000.08
Router(dhcp-config)# host 192.168.1.100 255.255.255.0   ! резервування адреси
Router(dhcp-config)# exit

! DHCP Relay — якщо сервер в іншій підмережі
Router(config-if)# ip helper-address 192.168.100.10
Router(config-if)# ip forward-protocol udp 67
Router(config-if)# ip forward-protocol udp 68

! Додаткові параметри DHCP
Router(config)# ip dhcp ping timeout 100
Router(config)# ip dhcp ping packets 2
Router(config)# ip dhcp conflict logging
Router(config)# ip dhcp snooping
```

### 14.3 DHCP Snooping

```
! Захист від підроблених DHCP-серверів
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10,20,30
Switch(config)# ip dhcp snooping information option
Switch(config)# ip dhcp snooping limit rate 100     ! макс. DHCP-пакетів на порту
Switch(config)# ip dhcp snooping verify mac-address
Switch(config)# ip dhcp snooping database flash:dhcp-snooping.db
Switch(config)# ip dhcp snooping database write-delay 300

! Довірений/ненадійний порт
Switch(config-if)# ip dhcp snooping trust           ! тільки uplink до реального сервера
Switch(config-if)# ip dhcp snooping limit rate 50
Switch(config-if)# ip dhcp snooping vlan 10 information option

! Перевірка
Switch# show ip dhcp snooping
Switch# show ip dhcp snooping binding
Switch# show ip dhcp snooping statistics
Switch# debug ip dhcp snooping
```

### 14.4 Налаштування ARP

```
! Налаштування ARP
Router(config)# arp timeout 300                             ! час зберігання ARP-запису
Router(config)# arp 192.168.1.100 0050.7966.6800 arpa      ! статичний ARP-запис
Router(config)# arp proxy disable
Router(config)# arp gratuitous ignore

! Proxy ARP
Router(config-if)# ip proxy-arp                            ! дозволити proxy ARP
Router(config-if)# no ip proxy-arp                         ! заборонити

! ARP Inspection (DAI) — захист від ARP-spoofing
Switch(config)# ip arp inspection vlan 10,20
Switch(config)# ip arp inspection validate src-mac dst-mac ip
Switch(config)# ip arp inspection log-buffer entries 32
Switch(config)# ip arp inspection log-buffer logs 1024 interval 10
Switch(config-if)# ip arp inspection trust                  ! довірений порт (uplink)
Switch(config-if)# ip arp inspection limit rate 100
```

---

## 15 Статична маршрутизація

### 15.1 Основи статичної маршрутизації

```
! Стандартні статичні маршрути
Router(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2              ! через next-hop
Router(config)# ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/1 10.0.0.3
Router(config)# ip route 192.168.4.0 255.255.255.0 10.0.0.4 50           ! AD=50

! Default Route (маршрут за замовчуванням)
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
Router(config)# ip route 0.0.0.0 0.0.0.0 Dialer1

! Summary Routes (зведені маршрути)
Router(config)# ip route 172.16.0.0 255.255.0.0 10.0.0.1
Router(config)# ip route 10.0.0.0 255.255.0.0 Null0                      ! "чорна діра"

! Floating Static Routes (резервний маршрут з більшою AD)
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.1                    ! AD=1 основний
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.2.1 250                ! AD=250 резервний
```

### 15.2 Відстеження статичних маршрутів

```
! IP SLA — зонд для відстеження досяжності
Router(config)# ip sla 1
Router(config-ip-sla)# icmp-echo 8.8.8.8 source-interface GigabitEthernet0/0
Router(config-ip-sla)# timeout 1000
Router(config-ip-sla)# frequency 5                                        ! ping кожні 5 сек
Router(config-ip-sla)# exit

Router(config)# ip sla schedule 1 life forever start-time now

! Track-об'єкт — реагує на стан SLA
Router(config)# track 10 ip sla 1 reachability
Router(config-track)# delay down 10 up 5

! Маршрут зникає, якщо track=down
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.1 track 10
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.2.1 250                ! резервний
```

### 15.3 Policy-Based Routing

```
! PBR — маршрутизація на основі правил (не тільки за dest. IP)
Router(config)# access-list 100 permit tcp any any eq 80
Router(config)# access-list 100 permit tcp any any eq 443

Router(config)# route-map PBR-OUTBOUND permit 10
Router(config-route-map)# match ip address 100                            ! яким трафіком
Router(config-route-map)# set ip next-hop 192.168.1.10                   ! куди направити
Router(config-route-map)# set interface Null0
Router(config-route-map)# set ip dscp af41
Router(config-route-map)# set ip precedence flash
```

---

> 📌 **Зберегти конфігурацію:** `copy running-config startup-config` або `wr`  
