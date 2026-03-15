# Cisco IOS — Шпаргалка команд
> Курс: Комп'ютерні мережі | 2 курс

---

## 1 Основи роботи

### 1.1 Скидання налаштувань

> ⚠️ Ці команди повністю видалять дані з пристрою. Завжди зберігайте резервні копії!

```
! Повне скидання (знищення всіх даних)
delete nvram:startup-config
delete flash:vlan.dat
erase nvram:
erase flash:
write erase
reload

! Часткове скидання (зі збереженням IOS)
write erase
delete vlan.dat
reload
```

### 1.2 Процес завантаження та ROMMON

```
! Перервати завантаження: натисніть Ctrl+Break протягом перших 60 секунд

! Команди ROMMON:
rommon 1 > confreg 0x2142                                   ! обхід завантаження startup-config
rommon 2 > reset                                            ! перезавантаження
rommon 3 > dir flash:                                       ! список файлів у Flash
rommon 4 > boot flash:c2900-universalk9-mz.SPA.157-3.M.bin  ! завантажити конкретний IOS
rommon 5 > tftpdnld                                         ! завантаження IOS з TFTP-сервера
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
show inventory                      ! перелік модулів та серійних номерів
show diag                           ! детальна діагностика
show environment                    ! температура, живлення, вентилятори
show module                         ! встановлені модулі
show idprom backplane

! Cisco 2960X/3650/3850 — команди для стекованих шасі
show switch                         ! список пристроїв у стеку
show switch detail                  ! детально
show switch neighbors               ! сусіди по стеку
switch 1 provision ws-c3650-24ps
switch 1 priority 15                ! більше значення = головний пристрій
switch 2 renumber 3
switch 1 reload
```

### 1.5 Управління прошивкою IOS

```
! Перевірити поточну версію IOS
show version           ! версія IOS, uptime, модель пристрою
show boot
show flash:
dir flash:

! Оновити IOS через TFTP
copy tftp://192.168.1.10/c2900-universalk9-mz.SPA.157-3.M.bin flash:
boot system flash:c2900-universalk9-mz.SPA.157-3.M.bin
copy running-config startup-config
reload

! Встановлення пакетів Bundle/Install (Cat3K/4K)
archive download-sw /force-reload /overwrite tftp://192.168.1.10/cat3k_caa-universalk9.16.12.04.SPA.bin

! Режим встановлення (IOS XE)
request platform software package install switch all file tftp://192.168.1.10/asr1000-universalk9.17.03.03.SPA.pkg
```

---

## 2 Навігація та основи IOS

### 2.1 Основи командного рядка

```
! Скорочення команд (до унікальних символів)
conf t          ! configure terminal
sh run          ! show running-config
sh ip int br    ! show ip interface brief
wr              ! write memory (copy run start)

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
show running-config | include ospf      ! рядки що містять "ospf"
show running-config | section router    ! секція "router"
show running-config | exclude !         ! без рядків-коментарів
show running-config | begin interface   ! починаючи з рядка "interface"
show running-config | count ospf        ! кількість входжень
```

### 2.2 Контекстна довідка

```
?                           ! список доступних команд
show ?                      ! параметри команди show
configure ?                 ! параметри configure
interface gigabitEthernet 0/0/0 ?
show ip route <Tab>         ! автодоповнення

! Часткове доповнення
sh ip ro<Tab>               ! доповнює до "show ip route"
```

### 2.3 Рівні привілеїв користувачів

```
! Налаштування рівнів привілеїв (0–15, де 15 = повний доступ)
privilege exec level 5 configure terminal
privilege exec level 5 show running-config
privilege interface level 5 shutdown
username admin privilege 5 secret P@ssw0rd

! Перегляд поточного рівня
show privilege
show privilege all

! Встановити пароль для рівня та увійти
enable secret level 5 SuperSecret123
enable 5
```

### 2.4 Псевдоніми команд

```
! Створення власних псевдонімів команд
alias exec srs show running-config | section
alias exec shbr show ip interface brief
alias exec routes show ip route
alias exec neighbors show cdp neighbors detail
alias configure config t

! Постійний псевдонім для резервного копіювання
alias exec backup copy running-config tftp://192.168.1.10/
backup R1-config.txt
```

### 2.5 Скрипти за допомогою TCL

```
! Вхід в TCL-оболонку для автоматизації команд
tclsh
 puts [exec "show version"]
 foreach i {1 2 3 4 5} { puts "Interface Gi0/$i" }
 exit

! Одноразова TCL-команда
tclsh puts [exec "show ip interface brief"]
```

---

## 3 Робота з файловою системою

### 3.1 Навігація файловою системою

```
! Навігація по файловій системі
pwd                     ! поточна директорія
dir                     ! список файлів
dir flash:              ! вміст Flash-пам'яті
dir nvram:              ! вміст NVRAM
dir usbflash0:          ! USB (якщо підключено)
dir sdflash:            ! SD-карта (ASR/ISR)
dir bootflash:          ! завантажувальна Flash
dir system:             ! системні файли

! Операції з файлами
copy source-url destination-url                 ! копіювати файл
copy running-config flash:backup-config         ! зберегти конфіг у Flash
copy flash:config.text tftp://192.168.1.10/     ! копіювати на TFTP
delete flash:old-config.text                    ! видалити файл
erase flash:                                    ! ⚠️ очистити всю Flash
mkdir flash:backups                             ! створити директорію
rmdir flash:backups                             ! видалити директорію
```

### 3.2 Управління конфігурацією

```
! Архівування конфігурацій
archive
  path flash:backups/$h-config  ! $h = hostname
  maximum 10                    ! зберігати до 10 версій
  time-period 1440              ! авто-збереження кожні 24 год
  write-memory                  ! архівувати при wr

! Перегляд та відновлення з архіву
show archive
configure replace flash:backups/R1-config-1  ! відкат до версії

! Відкат конфігурації
configure replace flash:backup-config 10
configure revert now
configure confirm

! Точки відновлення (IOS XE)
checkpoint database
show checkpoint database
rollback checkpoint checkpoint_name
```

### 3.3 Робота з SCP

```
! Налаштувати SCP-сервер на пристрої
ip scp server enable
username admin secret admin123
ip ssh version 2

! Копіювання через SCP (зашифровано, на відміну від TFTP)
copy running-config scp://admin@192.168.1.10/backups/R1-config
copy scp://admin@192.168.1.10/configs/new-config running-config

! З Linux на маршрутизатор
# scp R1-config admin@10.0.0.1:flash:
```

### 3.4 Робота з USB

```
! Перевірити стан USB
show usb
dir usbflash0:

! Копіювати на/з USB
copy running-config usbflash0:/backup-config
copy usbflash0:/new-ios.bin flash:

! Завантажитись з USB
boot system usbflash0:c2900-universalk9-mz.SPA.157-3.M.bin
```

### 3.5 Автоматизація бекапів

```
! EEM-скрипт для автоматичного резервного копіювання щодня о 02:00
event manager applet AUTO-BACKUP
  event timer cron cron-entry "0 2 * * *"
  action 1.0 cli command "enable"
  action 2.0 cli command "copy running-config tftp://192.168.1.10/backups/$_device_name-$_event_pub_time"
  action 3.0 syslog msg "Backup completed successfully"
```

---

## 4 Управління ліцензіями

### 4.1 Типи та моделі ліцензій

```
! Переглянути інформацію про ліцензії
show license
show license feature
show license udi                       ! унікальний ідентифікатор пристрою
show license right-to-use
show license tech-support

! Smart Licensing (IOS XE)
show license status
show license authorization
license smart register idtoken XXXX-XXXX-XXXX-XXXX
license smart reservation request local
license smart reservation install file flash:reservation.lic

! Traditional Licensing (класичний варіант)
show license all
license install flash:license.lic
license boot level ipbasek9
reload
```

### 4.2 Робота з ліцензіями

```
! Зберегти ліцензію у файл
license save flash:all_licenses.lic

! Експорт/імпорт ліцензії
license export flash:license_pa.lic
license import flash:new_license.lic

! Ознайомча (Evaluation) ліцензія
license accept end user agreement
license boot level securityk9
reload

! Перевірити відповідність ліцензій
show license violation
```

### 4.3 Right-to-Use (RTU) Ліцензування

```
! Управління RTU-ліцензіями
license right-to-use activate ipservices
license right-to-use deactivate ipservices
show license right-to-use
license right-to-use perpetual ipservices accept
```

---

## 5 Управління пристроєм

### 5.1 Системні налаштування

```
! Базове налаштування пристрою
hostname R1-CORE
no ip domain-lookup                 ! вимкнути DNS-запити в CLI
ip domain-name company.com
service password-encryption         ! шифрувати паролі у конфізі
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
logging buffered 16384 debugging
logging console critical
logging monitor debugging
logging host 192.168.100.10
logging source-interface Loopback0

! Налаштування банерів
banner motd ^
************************************************************
*              AUTHORIZED ACCESS ONLY                      *
*  This system is the property of Company Inc.             *
*Unauthorized access is prohibited and will be prosecuted. *
************************************************************
^

banner login ^
Please enter your credentials to access this device.
Contact Network Operations for assistance.
^

banner exec ^
Welcome to R1-CORE. You are logged in as $username.
Current time: $_timenow
^
```

### 5.2 Налаштування NTP

```
! Сервери NTP для синхронізації часу
ntp server 0.pool.ntp.org prefer
ntp server 1.pool.ntp.org
ntp server 2.pool.ntp.org
ntp server 192.168.100.10

! Автентифікація NTP
ntp authenticate
ntp authentication-key 1 md5 NTP-KEY-123
ntp trusted-key 1
ntp server 0.pool.ntp.org key 1

! NTP Master/Peer
ntp master 3               ! зробити пристрій NTP-сервером (stratum 3)
ntp peer 192.168.1.2
ntp update-calendar

! Контроль доступу до NTP
ntp access-group peer 10
ntp access-group serve-only 20
ntp access-group serve 30
ntp access-group query-only 40

! Перевірка
show ntp associations
show ntp status
show ntp clock
show clock detail
```

### 5.3 Налаштування DNS

```
! DNS-сервери
ip name-server 8.8.8.8
ip name-server 8.8.4.4
ip name-server 192.168.100.10
ip domain-name company.com
ip domain-list branch.company.com
ip domain-list partner.com
ip domain lookup source-interface Loopback0

! Статичні DNS-записи (hosts)
ip host router1.company.com 192.168.1.1
ip host switch1 10.0.0.1 10.0.0.2
ip host ftp-server 192.168.100.10

! DNS Caching
ip dns server
ip dns cache
ip dns spoofing 192.168.1.1

! Перевірка
show hosts
nslookup router1.company.com
ping router1.company.com
```

### 5.4 Налаштування Syslog

```
! Налаштування Syslog-сервера
logging on
logging host 192.168.100.10
logging host 192.168.100.11
logging trap debugging          ! рівень: debug/info/warning/error/critical
logging facility local7
logging source-interface Loopback0
logging origin-id hostname
logging sequence-numbers
logging discriminator OSPF msg-body includes "OSPF"

! Локальне логування (в буфер пристрою)
logging buffered 16384 informational
logging console warnings
logging monitor debugging
logging persistent url flash:logs size 4096
logging history debugging

! Фільтри логів
logging filter OSPF ROUTER-ID 1.1.1.1
logging filter BGP neighbor 192.168.1.2

! Перевірка
show logging
show logging history
show logging filter
```

### 5.5 Налаштування SNMP

```
! SNMPv2c (простий, без шифрування)
snmp-server community RO-COMMUNITY RO 10
snmp-server community RW-COMMUNITY RW 20
snmp-server host 192.168.100.10 version 2c RO-COMMUNITY
snmp-server enable traps
snmp-server trap-source Loopback0
snmp-server queue-length 100
snmp-server location "Data Center Rack A1"
snmp-server contact "Network Operations noc@company.com"
snmp-server chassis-id R1-CORE-ASR1001

! SNMPv3 (безпечний варіант з шифруванням)
snmp-server group ADMIN v3 priv read ALL write ALL
snmp-server user admin1 ADMIN v3 auth sha AuthPass123 priv aes 128 PrivPass123
snmp-server host 192.168.100.10 version 3 priv admin1
snmp-server enable traps snmp authentication linkdown coldstart warmstart
snmp-server enable traps config
snmp-server enable traps entity
snmp-server enable traps cpu threshold
snmp-server enable traps memory threshold
snmp-server enable traps ospf
snmp-server enable traps bgp

! SNMP Views (обмеження видимих OID)
snmp-server view ALL iso included
snmp-server view RESTRICTED system included
snmp-server view RESTRICTED interfaces excluded

! Перевірка
show snmp
show snmp user
show snmp group
show snmp host
```

---

## 6 Основи безпеки

### 6.1 Управління паролями

```
! Налаштування паролів
enable secret SuperSecret123!
enable algorithm-type scrypt secret EvenMoreSecret456!  ! сильніше хешування
username admin privilege 15 secret P@ssw0rd123!
username operator privilege 5 secret 0p3rator!
service password-encryption                             ! шифрувати всі паролі у конфізі
security passwords min-length 10                        ! мінімальна довжина пароля
login block-for 300 attempts 3 within 60                ! блокувати після 3 невдалих спроб за 60 сек
login delay 2                                           ! затримка між спробами входу
login on-failure log
login on-success log

! Налаштування ліній (console/vty)
Router(config-line)# password Cons0leP@ss
Router(config-line)# login local
Router(config-line)# exec-timeout 5 0                   ! автовихід через 5 хв бездіяльності
Router(config-line)# absolute-timeout 30                ! примусовий вихід через 30 хв
Router(config-line)# session-timeout 30
Router(config-line)# logout-warning 5

! Заборона відновлення пароля
no service password-recovery
enable secret recovery SecretRecovery789!
```

### 6.2 Налаштування SSH

```
! Налаштування SSH v2
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 2
ip ssh maxstartups 3
ip ssh logging events
ip ssh server algorithm mac hmac-sha2-256 hmac-sha2-512
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh server algorithm hostkey rsa-sha2-256 rsa-sha2-512
ip ssh server disable-deprecated-version 1.99
ip ssh server max-sessions-per-connection 10

! Автентифікація за ключем (замість пароля)
crypto key generate rsa modulus 4096
crypto key generate dsa modulus 2048
crypto key generate ecdsa curve 256
ip ssh pubkey-chain
Router(config-ssh-pubkey)# username admin
Router(config-ssh-pubkey)# key-hash ssh-rsa AAAA...==
Router(config-ssh-pubkey)# exit

! Source Interface для SSH
ip ssh source-interface Loopback0
ip ssh vrf MGMT

! Перевірка
show ip ssh
show ssh
show crypto key mypubkey rsa
```

### 6.3 Сервіси TCP/UDP

```
! Вимкнути непотрібні сервіси (hardening)
no service tcp-small-servers
no service udp-small-servers
no service finger
no ip bootp server
no ip http server
no ip http secure-server
no ip source-route
no ip gratuitous-arps
no cdp run                  ! вимкнути CDP глобально
no lldp run                 ! вимкнути LLDP глобально
no ip proxy-arp
no ip unreachables
no ip redirects
no ip mask-reply

! Вимкнути на конкретному інтерфейсі
 no cdp enable
 no lldp transmit
 no lldp receive
 no ip directed-broadcast
 no ip unreachables
 no ip proxy-arp
```

### 6.4 Політики Control Plane (CoPP)

```
! CoPP — обмежує трафік до CPU маршрутизатора
class-map match-any COPP-CRITICAL
Router(config-cmap)# match access-group 110
Router(config-cmap)# exit

class-map match-any COPP-IMPORTANT
Router(config-cmap)# match access-group 120
Router(config-cmap)# exit

policy-map COPP-POLICY
 class COPP-CRITICAL
  police 8000 conform-action transmit exceed-action drop
  exit
 class COPP-IMPORTANT
  police 40000 conform-action transmit exceed-action drop
  exit
 class class-default
  police 10000 conform-action transmit exceed-action drop
  exit
 exit

control-plane
 service-policy input COPP-POLICY
 exit

! ACL для CoPP
access-list 110 permit tcp any any eq 22
access-list 110 permit tcp any any eq 23
access-list 120 permit ospf any any
access-list 120 permit eigrp any any
access-list 120 permit pim any any
```

---

## 7 AAA & Управління доступом

### 7.1 Основи AAA

```
! Увімкнути AAA (Authentication, Authorization, Accounting)
aaa new-model
aaa authentication login default local
aaa authentication enable default enable
aaa authorization exec default local
aaa accounting exec default start-stop group radius

! Локальна база користувачів
username admin privilege 15 algorithm-type scrypt secret AdminPass123!
username operator privilege 5 secret 0p3rator!
username guest privilege 1 secret GuestAccess!

! Паролі для рівнів enable
enable secret level 15 SuperSecret!
enable secret level 5 OperatorPass
```

### 7.2 Налаштування RADIUS

```
! Налаштування RADIUS-серверів
radius server ISE-PRIMARY
 address ipv4 192.168.100.10 auth-port 1812 acct-port 1813
 key RadiusKey123!
 retransmit 3
 timeout 5
 exit

radius server ISE-SECONDARY
 address ipv4 192.168.100.11 auth-port 1812 acct-port 1813
 key RadiusKey456!
 exit

! Група серверів RADIUS
aaa group server radius ISE-GROUP
Router(config-sg-radius)# server name ISE-PRIMARY
Router(config-sg-radius)# server name ISE-SECONDARY
Router(config-sg-radius)# exit

! AAA з RADIUS
aaa authentication login default group ISE-GROUP local
aaa authentication enable default group ISE-GROUP enable
aaa authorization exec default group ISE-GROUP local
aaa accounting exec default start-stop group ISE-GROUP
aaa accounting network default start-stop group ISE-GROUP
```

### 7.3 Налаштування TACACS+

```
! Налаштування TACACS+-серверів
tacacs server TACACS-PRIMARY
 address ipv4 192.168.100.20
 key TacacsKey123!
 single-connection
 timeout 10
 exit

! Група серверів TACACS+
aaa group server tacacs+ TACACS-GROUP
 server name TACACS-PRIMARY
 exit

! AAA з TACACS+
aaa authentication login default group TACACS-GROUP local
aaa authentication enable default group TACACS-GROUP enable
aaa authorization commands 15 default group TACACS-GROUP local
aaa authorization config-commands
aaa authorization exec default group TACACS-GROUP local
aaa accounting commands 15 default start-stop group TACACS-GROUP
aaa accounting exec default start-stop group TACACS-GROUP
```

### 7.4 Авторизація команд

```
! Рівні авторизації команд
privilege exec level 5 show running-config
privilege exec level 5 show interfaces
privilege exec level 10 configure terminal
privilege exec level 15 reload
privilege interface level 5 shutdown
privilege interface level 10 ip address

! CLI Views — набори дозволених команд для різних ролей
parser view ROOT
 secret RootView123!
 commands exec include all show
 commands exec include configure
 commands exec include reload
 exit

parser view OPERATOR
 secret OperatorView456!
 commands exec include show
 commands exec exclude show running-config  ! заборонити конкретну команду
 commands exec include ping
 commands exec include traceroute
 exit
```

### 7.5 Downloadable ACLs

```
! ACL, що завантажується з RADIUS після автентифікації
ip access-list extended DYNACL-WEBSERVER
 permit tcp any host 192.168.1.10 eq 80
 permit tcp any host 192.168.1.10 eq 443
 deny ip any any
 exit

! AAA атрибут для ACL
aaa authorization network default group ISE-GROUP
```

---

## 8 Протоколи управління

### 8.1 Управління HTTP/HTTPS

```
! Налаштування HTTP/HTTPS для веб-інтерфейсу
ip http server
ip http port 8080
ip http authentication local
ip http access-class 10            ! обмежити доступ ACL
ip http secure-server
ip http secure-port 443
ip http secure-ciphersuite aes256-cbc-sha1
ip http timeout-policy idle 60 life 86400 requests 100

! ACL для обмеження доступу до HTTP
access-list 10 permit 192.168.100.0 0.0.0.255
access-list 10 deny any

! Перевірка
show ip http server status
show ip http secure-server status
```

### 8.2 Netconf/RESTCONF

```
! NETCONF — програмне управління пристроєм (через SSH, порт 830)
netconf-yang
netconf-yang feature candidate-datastore
netconf-yang feature notify
netconf-yang feature url
netconf-yang ssh port 830

! RESTCONF — HTTP-API для управління пристроєм
restconf
ip http secure-server
restconf data-entries 10000

! Перевірка
show netconf-yang sessions
show restconf
```

### 8.3 gRPC/Telemetry

```
! gRPC — протокол для streaming-телеметрії
grpc
 port 57400
 no-tls
 address-family ipv4
 max-request-total 64
 max-request-per-user 16

! Телеметрія — потокова передача метрик на колектор
telemetry ietf subscription 101
 encoding encode-kvgpb
 filter xpath /interfaces/interface/statistics
 source-address 192.168.1.1
 stream yang-push
 update-policy periodic 500
 receiver ip address 192.168.100.30 5432 protocol grpc-tcp
```

### 8.4 NETCONF/YANG моделі

```
! Перевірка YANG-моделей
show platform software yang-management process
show yang-operational memory
show yang schema

! Встановлення YANG-моделей
copy tftp://192.168.1.10/ietf-interfaces@2014-05-08.yang flash:
install add file flash:ietf-interfaces@2014-05-08.yang activate

! NETCONF-операції (з Linux)
# netconf-console --host 10.0.0.1 --port 830 --user admin --password pass --get-config
```

---

## 9 Комутація Ethernet

### 9.1 Таблиця MAC-адрес

```
! Переглянути таблицю MAC-адрес
show mac address-table
show mac address-table dynamic
show mac address-table aging-time
show mac address-table count
show mac address-table interface gigabitEthernet 1/0/1
show mac address-table vlan 10
show mac address-table address 0050.7966.6800

! Налаштування таблиці MAC
mac address-table aging-time 300                                   ! час старіння запису (сек)
mac address-table learning vlan 10                                 ! дозволити навчання
no mac address-table learning vlan 20                              ! заборонити навчання
mac address-table static 0050.7966.6800 vlan 10 interface gi1/0/1  ! статичний запис
mac address-table notification change
mac address-table limit maximum 1000 vlan 10

! Очистити таблицю MAC
clear mac address-table dynamic
clear mac address-table dynamic interface gi1/0/1
clear mac address-table dynamic vlan 10
clear mac address-table dynamic address 0050.7966.6800
```

### 9.2 Налаштування портів

```
! Базове налаштування порту
interface gigabitEthernet 1/0/1
 description "Connected to Server01"
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 spanning-tree portfast           ! прискорити вхід у forwarding
 spanning-tree bpduguard enable   ! захист від несанкціонованих SW
 no shutdown
 exit

! Швидкість та дуплекс
 speed 1000
 duplex full
 negotiation auto
 mtu 9216                         ! jumbo frames

! Відновлення портів з err-disabled стану
errdisable recovery cause psecure-violation
errdisable recovery cause bpduguard
errdisable recovery interval 30
errdisable detect cause all

! Шаблони інтерфейсів (застосувати однакові налаштування до багатьох портів)
template USER-PORT
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 storm-control broadcast level 10.00
 service-policy input AUTOQOS-VOIP
 exit

interface range gi1/0/1 - 24
 source template USER-PORT
```

### 9.3 CDP/LLDP

```
! CDP — виявлення сусідніх Cisco-пристроїв
cdp run
cdp timer 60                        ! інтервал надсилання (сек)
cdp holdtime 180                    ! час утримання інформації
cdp advertise-v2
 cdp enable
 no cdp enable                      ! вимкнути на порту

! LLDP — стандартний протокол виявлення (будь-який виробник)
lldp run
lldp timer 30
lldp holdtime 120
lldp reinit 2
 lldp transmit
 lldp receive
 lldp med

! Перевірка
show cdp neighbors
show cdp neighbors detail
show cdp traffic
show lldp neighbors
show lldp neighbors detail
show lldp traffic
show lldp interface gi1/0/1
```

### 9.4 UDLD & Loop Guard

```
! UDLD — виявлення однонаправлених зв'язків
udld enable
udld aggressive              ! агресивний режим: блокує порт при збої
 udld port aggressive
 udld disable

! Loop Guard — захист від петель через втрату BPDU
spanning-tree loopguard default
 spanning-tree guard loop
 spanning-tree guard root

! Перевірка
show udld neighbors
show udld gi1/0/1
show spanning-tree inconsistentports
```

---

## 10 VLANs & Trunking

### 10.1 Налаштування VLAN

```
! Створення VLAN
vlan 10
 name SALES
 state active
 exit

vlan 20
 name ENGINEERING
 exit

vlan 99
 name MANAGEMENT
 exit

! Діапазон VLAN
vlan 100-200
 name USER-VLANS
 exit

! Extended VLANs (1006–4094)
vlan 2000
 name EXTENDED-VLAN
 exit

! Private VLANs — ізоляція в межах VLAN
vlan 500
 private-vlan primary
 private-vlan association 501-502
 exit

vlan 501
 private-vlan isolated          ! порти не бачать одне одного
 exit

vlan 502
 private-vlan community         ! порти бачать одне одного
 exit
```

### 10.2 VLAN Trunking

```
! Налаштування транк-порту (передає кілька VLAN)
interface gigabitEthernet 1/0/24
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99       ! нетегований VLAN
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk allowed vlan add 40-50
 switchport trunk allowed vlan remove 30
 switchport trunk pruning vlan 2-1001
 switchport nonegotiate               ! вимкнути DTP
 spanning-tree guard root
 exit

! Dynamic Trunking Protocol (DTP)
 switchport mode dynamic desirable     ! активно ініціює транк
 switchport mode dynamic auto          ! чекає ініціативи

! VLAN Translation (перетворення номерів VLAN)
 switchport vlan mapping 10 110
 switchport vlan mapping 20 120

! Перевірка
show interfaces trunk
show interfaces gi1/0/24 switchport
show interfaces gi1/0/24 trunk
show vlan
show vlan id 10
show vlan name SALES
```

### 10.3 Налаштування VTP

```
! VTP — синхронізація VLAN між комутаторами
vtp domain COMPANY
vtp mode server      ! server = створює/змінює VLAN
vtp password VTP-Pass123!
vtp version 2
vtp pruning          ! не надсилати трафік VLAN туди, де їх немає
vtp interface Loopback0

! VTP Client (отримує VLAN від server, не може створювати)
vtp mode client
vtp transparent      ! не бере участі в VTP, але пропускає його

! Перевірка
show vtp status
show vtp password
show vtp counters
show vlan brief
```

### 10.4 Voice VLAN

```
! Налаштування Voice VLAN для IP-телефонів
interface gigabitEthernet 1/0/10
 switchport voice vlan 110
 switchport priority extend cos 0
 auto qos voip trust
 mls qos trust cos              ! довіряти CoS-мітці від телефону
 spanning-tree portfast
 spanning-tree bpduguard enable

! LLDP-MED для IP-телефонів
lldp run
 lldp med
 lldp med-tlv-select inventory-management network-policy
```

### 10.5 VLAN ACLs (VACLs)

```
! VACL — фільтрація трафіку всередині VLAN
vlan access-map BLOCK-SERVER 10
 match ip address SERVER-ACL
 action drop
 exit

vlan access-map BLOCK-SERVER 20
 action forward
 exit

vlan filter BLOCK-SERVER vlan-list 10  ! застосувати до VLAN 10

! ACL для VACL
ip access-list extended SERVER-ACL
 deny tcp any host 192.168.10.10 eq 3389
 permit ip any any
```

---

## 11 Spanning Tree Protocol

### 11.1 Основи налаштування STP

```
! Вибір режиму STP
spanning-tree mode rapid-pvst    ! Rapid PVST+ — рекомендовано
spanning-tree mode mst           ! MST — для великих мереж
spanning-tree mode pvst          ! класичний PVST

! Пріоритет мосту (менше = більше шансів стати Root Bridge)
spanning-tree vlan 1,10,20,30 priority 4096
spanning-tree vlan 40-50 priority 8192

! Захист Root Bridge
spanning-tree portfast bpduguard default
 spanning-tree guard root
 spanning-tree bpdufilter enable
 spanning-tree bpduguard enable

! Перевірка
show spanning-tree
show spanning-tree vlan 10
show spanning-tree interface gi1/0/1
show spanning-tree detail
show spanning-tree inconsistentports
show spanning-tree mst configuration
```

### 11.2 Налаштування MST

```
! Налаштування MST-регіону
spanning-tree mst configuration
 name REGION1
 revision 1
 instance 1 vlan 10,20,30    ! прив'язати VLAN до instance
 instance 2 vlan 40,50,60
 exit

! Параметри MST-instance
spanning-tree mst 1 priority 4096
spanning-tree mst 2 priority 8192
spanning-tree mst 0-2 hello-time 2
spanning-tree mst 0-2 forward-time 15
spanning-tree mst 0-2 max-age 20

! Перевірка
show spanning-tree mst
show spanning-tree mst configuration
show spanning-tree mst 1
show spanning-tree mst interface gi1/0/1
```

### 11.3 Таймери та оптимізація STP

```
! Таймери STP
spanning-tree vlan 1 hello-time 2      ! BPDU кожні 2 сек
spanning-tree vlan 1 forward-time 15   ! затримка переходу
spanning-tree vlan 1 max-age 20        ! час до визнання Root недоступним
spanning-tree transmit hold-count 6

! PortFast & BPDU Guard
spanning-tree portfast default         ! увімкнути глобально для access-портів
spanning-tree portfast bpduguard default
 spanning-tree portfast
 spanning-tree portfast trunk

! UplinkFast & BackboneFast (тільки для PVST)
spanning-tree uplinkfast               ! швидший переключення uplink
spanning-tree backbonefast             ! швидше виявлення змін топології

! Loop Guard
spanning-tree loopguard default
 spanning-tree guard loop
```

### 11.4 Безпека STP

```
! BPDU Guard — вимкнути порт якщо прийшов BPDU (захист від зайвих SW)
spanning-tree portfast bpduguard default
errdisable recovery cause bpduguard
errdisable recovery interval 30

! BPDU Filter — не надсилати та не обробляти BPDU на порту
spanning-tree portfast bpdufilter default
 spanning-tree bpdufilter enable

! Root Guard — заборонити порту ставати root port
 spanning-tree guard root

! TC Guard — обмежити обробку Topology Change повідомлень
spanning-tree tc-guard
spanning-tree tc-filter

! Перевірка
show spanning-tree summary
show spanning-tree detail
debug spanning-tree events
```

---

## 12 EtherChannel & LACP

### 12.1 Налаштування EtherChannel

```
! LACP Configuration (рекомендовано)
interface range gigabitEthernet 1/0/1-2
 channel-group 1 mode active    ! активно надсилає LACP PDU
 channel-protocol lacp
 lacp port-priority 1000        ! менше = вищий пріоритет
 lacp rate fast
 exit

! PAgP Configuration (протокол Cisco)
 channel-group 1 mode desirable
 channel-protocol pagp
 pagp port-priority 1000

! Static EtherChannel (без протоколу, обидва боки = on)
 channel-group 1 mode on

! Port-Channel Interface — логічний інтерфейс для групи
interface port-channel 1
 description "Uplink to Core"
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 lacp system-priority 1000
 lacp system-id 0050.7966.6800
 exit
```

### 12.2 Балансування навантаження

```
! Алгоритми балансування трафіку по каналах
port-channel load-balance src-dst-ip    ! за IP-адресами
port-channel load-balance src-dst-mac   ! за MAC-адресами
port-channel load-balance src-dst-port  ! за TCP/UDP портами
port-channel load-balance src-ip
port-channel load-balance dst-ip
port-channel load-balance src-mac
port-channel load-balance dst-mac

! Балансування на рівні VLAN
vlan 10
 lacp load-balancing src-dst-ip
 exit

! Перевірка
show etherchannel summary
show etherchannel port-channel
show etherchannel load-balance
show lacp neighbor
show lacp internal
show lacp counters
show pagp neighbor
show pagp internal
```

### 12.3 Розширене налаштування EtherChannel

```
! Cross-Stack EtherChannel (StackWise)
interface port-channel 10
 stack-port 1/1-1/2
 switchport mode trunk
 exit

! VSS EtherChannel
interface port-channel 100
 vss-port-channel
 switchport mode trunk
 exit

! Мінімальна кількість активних портів
 lacp min-links 2
port-channel min-links 2

! Максимальна кількість активних портів у bundle
 lacp max-bundle 8

! Graceful Shutdown
 lacp graceful-convergence
```

### 12.4 Пошук несправностей в EtherChannel

```
! Діагностика проблем
show etherchannel detail
show etherchannel inconsistency  ! виявити розбіжності в налаштуваннях
debug etherchannel
debug lacp
debug pagp

! Скинути лічильники
clear lacp counters
clear pagp counters
default interface port-channel 1  ! скинути налаштування port-channel

! Перенастройка порту вручну
interface gi1/0/1
 shutdown
 no channel-group
 no shutdown
 channel-group 1 mode active
```

---

## 13 IP адресація

### 13.1 Налаштування IP-адрес

```
! Базова IP-адресація
interface gigabitEthernet 0/0
 ip address 192.168.1.1 255.255.255.0
 ip address 10.0.0.1 255.255.255.0 secondary  ! другорядна адреса
 ip directed-broadcast
 ip unreachables
 ip redirects
 ip proxy-arp
 no shutdown

! Loopback-інтерфейс (завжди активний, використовується як Router ID)
interface loopback 0
 ip address 1.1.1.1 255.255.255.255
 description "Router ID and Management"
 ip ospf network point-to-point

! VLSM — маски змінної довжини
 ip address 172.16.1.1 255.255.255.192    ! /26
 ip address 10.1.1.1 255.255.255.224      ! /27
 ip address 192.168.1.129 255.255.255.240 ! /28

! IPv6
 ipv6 enable
 ipv6 address 2001:db8::1/64
 ipv6 address fe80::1 link-local
 ipv6 address autoconfig
```

### 13.2 Сервіси DHCP

```
! DHCP-сервер
ip dhcp excluded-address 192.168.1.1 192.168.1.10  ! резерв для статики
ip dhcp excluded-address 192.168.1.254
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8 8.8.4.4
 domain-name company.com
 lease 8                            ! оренда 8 днів
 option 150 ip 192.168.1.10         ! TFTP для VoIP
 client-identifier 0100.5056.c000.08
 host 192.168.1.100 255.255.255.0   ! резервування адреси
 exit

! DHCP Relay — якщо сервер в іншій підмережі
 ip helper-address 192.168.100.10
 ip forward-protocol udp 67
 ip forward-protocol udp 68

! Додаткові параметри DHCP
ip dhcp ping timeout 100
ip dhcp ping packets 2
ip dhcp conflict logging
ip dhcp snooping
```

### 13.3 DHCP Snooping

```
! Захист від підроблених DHCP-серверів
ip dhcp snooping
ip dhcp snooping vlan 10,20,30
ip dhcp snooping information option
ip dhcp snooping limit rate 100     ! макс. DHCP-пакетів на порту
ip dhcp snooping verify mac-address
ip dhcp snooping database flash:dhcp-snooping.db
ip dhcp snooping database write-delay 300

! Довірений/ненадійний порт
 ip dhcp snooping trust             ! тільки uplink до реального сервера
 ip dhcp snooping limit rate 50
 ip dhcp snooping vlan 10 information option

! Перевірка
show ip dhcp snooping
show ip dhcp snooping binding
show ip dhcp snooping statistics
debug ip dhcp snooping
```

### 13.4 Налаштування ARP

```
! Налаштування ARP
arp timeout 300                            ! час зберігання ARP-запису
arp 192.168.1.100 0050.7966.6800 arpa      ! статичний ARP-запис
arp proxy disable
arp gratuitous ignore

! Proxy ARP
 ip proxy-arp                            ! дозволити proxy ARP
 no ip proxy-arp                         ! заборонити

! ARP Inspection (DAI) — захист від ARP-spoofing
ip arp inspection vlan 10,20
ip arp inspection validate src-mac dst-mac ip
ip arp inspection log-buffer entries 32
ip arp inspection log-buffer logs 1024 interval 10
 ip arp inspection trust                 ! довірений порт (uplink)
 ip arp inspection limit rate 100
```

---

## 14 Статична маршрутизація

### 14.1 Основи статичної маршрутизації

```
! Стандартні статичні маршрути
ip route 192.168.2.0 255.255.255.0 10.0.0.2             ! через next-hop
ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/1 10.0.0.3
ip route 192.168.4.0 255.255.255.0 10.0.0.4 50          ! AD=50

! Default Route (маршрут за замовчуванням)
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 Dialer1

! Summary Routes (зведені маршрути)
ip route 172.16.0.0 255.255.0.0 10.0.0.1
ip route 10.0.0.0 255.255.0.0 Null0                     ! "чорна діра"

! Floating Static Routes (резервний маршрут з більшою AD)
ip route 0.0.0.0 0.0.0.0 192.168.1.1                    ! AD=1 основний
ip route 0.0.0.0 0.0.0.0 192.168.2.1 250                ! AD=250 резервний
```

### 14.2 Відстеження статичних маршрутів

```
! IP SLA — зонд для відстеження досяжності
ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet0/0
 timeout 1000
 frequency 5                                        ! ping кожні 5 сек
 exit

ip sla schedule 1 life forever start-time now

! Track-об'єкт — реагує на стан SLA
track 10 ip sla 1 reachability
 delay down 10 up 5

! Маршрут зникає, якщо track=down
ip route 0.0.0.0 0.0.0.0 192.168.1.1 track 10
ip route 0.0.0.0 0.0.0.0 192.168.2.1 250                ! резервний
```

### 14.3 Policy-Based Routing

```
! PBR — маршрутизація на основі правил (не тільки за dest. IP)
access-list 100 permit tcp any any eq 80
access-list 100 permit tcp any any eq 443

route-map PBR-OUTBOUND permit 10
 match ip address 100                           ! яким трафіком
 set ip next-hop 192.168.1.10                   ! куди направити
 set interface Null0
 set ip dscp af41
 set ip precedence flash
```

---

> 📌 **Зберегти конфігурацію:** `copy running-config startup-config` або `wr`  
