# Інструкція з впровадження правил безпеки мережевих пристроїв
## (Cisco IOS/IOS-XE - комутатори та маршрутизатори; MikroTik RouterOS — маршрутизатори)
 
Документ описує базовий набір заходів по захисту самих мережевих пристроїв (device hardening)
 
---
 
# ЧАСТИНА 1. CISCO IOS / IOS-XE
 
## 1.1. Захист віртуальних підключень (VTY, console, aux)
 
### 1.1.1. Обмеження доступу до VTY по ACL
 
```
ip access-list standard MGMT-ACCESS
 permit 10.90.30.0 0.0.0.255
 permit 10.90.40.5
 deny any log
 
line vty 0 15
 access-class MGMT-ACCESS in
 transport input ssh
 exec-timeout 5 0
 logging synchronous
```
 
- `access-class` дозволяє підключення лише з довірених підмереж/адрес
- `transport input ssh` вимикає telnet на VTY (детальніше — п. 1.5)
- `exec-timeout 5 0` — автоматичне завершення неактивної сесії (5 хв)
### 1.1.2. Консольний порт
 
```
line console 0
 exec-timeout 5 0
 password 7 <encrypted>
 login local
 logging synchronous
```
 
### 1.1.3. Обов’язкове шифрування SSH
 
```
ip domain-name bek.k21
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
```
 
### 1.1.4. Захист паролів у конфігурації
 
```
service password-encryption
enable secret <strong-password>
no enable password
```
 
`enable secret` використовує хешування (MD5/scrypt залежно від версії), на відміну від зворотно-оборотного `enable password`
 
---
 
## 1.2. Створення користувачів
 
```
username netadmin privilege 15 secret <strong-password>
username teacher privilege 5 secret <strong-password>
username auditor privilege 1 secret <strong-password>
 
line vty 0 15
 login local
```
 
Рекомендації:
- Кожен адміністратор - окремий обліковий запис (без спільних логінів)
- Використовувати `secret`, а не `password` (хешування замість plaintext/оборотного шифру)
- Мінімальна довжина паролів:
```
security passwords min-length 12
```
 
- Блокування облікового запису за перевищення часу бездіяльності неможливо в IOS напряму без зовнішнього AAA, тому контроль здійснюється через `login block-for` (див. п. 1.3) - він діє на рівні пристрою, а не окремого користувача
---
 
## 1.3. Налаштування "fail-to-ban" (захист від підбору пароля)
 
Cisco IOS має вбудований механізм `login block-for`, що тимчасово блокує спроби входу після перевищення порогу невдалих автентифікацій - аналог fail2ban на рівні пристрою
 
```
login block-for 120 attempts 4 within 60
login quiet-mode access-class QUIET-PERMIT
login delay 2
 
login on-failure log every 1
login on-success log every 1
 
ip access-list standard QUIET-PERMIT
 permit 10.90.40.5
```
 
Пояснення:
- `block-for 120 attempts 4 within 60` - якщо протягом 60 секунд відбулося 4 невдалі спроби входу (з будь-якої адреси), пристрій на 120 секунд блокує **всі** нові спроби автентифікації (SSH/Telnet/HTTP), крім адрес у `QUIET-PERMIT` (аварійний доступ для адміністратора)
- `login delay 2` - пауза 2 секунди між спробами введення пароля (уповільнює брутфорс)
- `login on-failure/on-success log` - журналювання спроб для подальшого аналізу (syslog)
Перегляд стану:
```
show login
show login failures
```
 
---
 
## 1.4. Призначення адміністративних ролей
 
### 1.4.1. Класичні рівні привілеїв (0–15)
 
```
privilege exec level 5 show running-config
privilege exec level 5 configure terminal
privilege exec level 1 show interfaces
 
username operator1 privilege 5 secret <password>
```
 
Рівень 15 - повний доступ, 1 - базовий read-only, проміжні рівні (2–14) налаштовуються під конкретні команди.
 
### 1.4.2. Parser Views (гнучкіший рольовий доступ, IOS 12.3+ / IOS-XE)
 
```
aaa new-model
enable secret <root-secret>
 
parser view NETWORK_OPERATOR
 secret <view-secret>
 commands exec include show
 commands exec include ping
 commands exec include traceroute
 commands configure include interface
 
parser view SECURITY_ADMIN
 secret <view-secret>
 commands exec include all show
 commands configure include all
```
 
> Примітка: `aaa new-model` тут вмикається **лише** для локальної рольової моделі (`parser view`) і роботи з локальною базою користувачів через `login local` - зовнішній AAA-сервер (RADIUS/TACACS+) при цьому не використовується і не налаштовується.
 
Активація ролі оператором:
```
enable view NETWORK_OPERATOR
```
 
---
 
## 1.5. Деактивація застарілих/небезпечних методів підключення
 
```
! Заборона Telnet — лише SSH
line vty 0 15
 transport input ssh
 
! Вимкнення HTTP-сервера керування (незашифрований)
no ip http server
ip http secure-server
ip http secure-port 443
ip http authentication local
 
! Вимкнення непотрібних "дрібних" сервісів
no service tcp-small-servers
no service udp-small-servers
no ip bootp server
no ip finger
no ip identd
 
! Вимкнення CDP там, де він не потрібен (за замовчуванням розкриває топологію)
no cdp run
! або точково на порт:
interface range Ethernet0/0-4
 no cdp enable
 
! Вимкнення застарілого/непотрібного LLDP на зовнішніх/user-портах
no lldp run
 
! Заборона SNMP v1/v2 (незашифровані community-рядки), використання лише v3
no snmp-server community public
no snmp-server community private
snmp-server group SNMPV3GROUP v3 priv
snmp-server user snmpadmin SNMPV3GROUP v3 auth sha <auth-pass> priv aes 128 <priv-pass>
 
! Забороняємо proxy-arp, source-routing тощо на інтерфейсах керування
interface Vlan1
 no ip proxy-arp
no ip source-route
 
! Банер попередження про несанкціонований доступ
banner login ^C
УВАГА! Доступ дозволено лише авторизованим користувачам.
Усі дії журналюються.
^C
```
 
---
 
# ЧАСТИНА 2. MIKROTIK ROUTEROS
 
## 2.1. Захист віртуальних підключень
 
### 2.1.1. Обмеження служб керування за адресою джерела
 
```
/ip service
set winbox address=10.90.40.0/24
set ssh address=10.90.40.0/24
set www disabled=yes
set www-ssl disabled=yes
set api disabled=yes
set api-ssl disabled=yes
set telnet disabled=yes
set ftp disabled=yes
```
 
### 2.1.2. Захист через firewall input-ланцюг (обов'язково поряд з `service address`)
 
```
/ip firewall filter
add chain=input protocol=tcp dst-port=22,8291 src-address-list=MGMT_ALLOWED action=accept comment="allow mgmt SSH/WinBox"
add chain=input protocol=tcp dst-port=22,8291 action=drop comment="drop other SSH/WinBox"
add chain=input connection-state=established,related action=accept
add chain=input connection-state=invalid action=drop
add chain=input action=drop comment="default deny to router itself"
 
/ip firewall address-list
add address=10.90.40.0/24 list=MGMT_ALLOWED
```
 
### 2.1.3. Таймаути неактивних сесій
 
```
/ip ssh
set forwarding-enabled=no strong-crypto=yes
 
/system console
# для serial/консолі — обмежити фізичний доступ організаційно
 
/ip service
set ssh port=2222   ;# зміна стандартного порту (security through obscurity, додатковий бар'єр)
```
 
---
 
## 2.2. Створення користувачів
 
```
/user group
add name=readonly policy=read,test,winbox
add name=netops policy=read,write,test,winbox,sniff
add name=fulladmin policy=local,telnet,ssh,ftp,reboot,read,write,policy,test,winbox,password,web,sniff,sensitive,api,romon,dude,tikapp
 
/user
add name=netadmin group=netops password=<strong-password>
remove [find name=admin]
```
 
Рекомендації:
- Обов'язково видалити або перейменувати/деактивувати обліковий запис `admin` за замовчуванням
- Кожен адміністратор - окремий обліковий запис
- Вимога складних паролів:
```
/user aaa
set minimum-password-length=12
```
 
---
 
## 2.3. Налаштування "fail-to-ban" (захист від brute-force)
 
RouterOS не має вбудованого `login block-for`, як Cisco, тому механізм реалізується через firewall (address-list з таймаутом — фактичний аналог fail2ban):
 
```
/ip firewall filter
 
# Крок 1: занесення адреси в чорний список після 3-ї невдалої спроби SSH за 5 хв
add chain=input protocol=tcp dst-port=2222 connection-state=new \
    src-address-list=ssh_stage3 action=add-src-to-address-list \
    address-list=ssh_blacklist address-list-timeout=1d comment="stage3->blacklist"
 
add chain=input protocol=tcp dst-port=2222 connection-state=new \
    src-address-list=ssh_stage2 action=add-src-to-address-list \
    address-list=ssh_stage3 address-list-timeout=1m
 
add chain=input protocol=tcp dst-port=2222 connection-state=new \
    src-address-list=ssh_stage1 action=add-src-to-address-list \
    address-list=ssh_stage2 address-list-timeout=1m
 
add chain=input protocol=tcp dst-port=2222 connection-state=new \
    action=add-src-to-address-list address-list=ssh_stage1 \
    address-list-timeout=1m
 
# Крок 2: блокування всіх, хто в blacklist
add chain=input src-address-list=ssh_blacklist action=drop comment="drop brute-force sources"
```
 
Логіка: кожна нова спроба підключення до SSH-порту переводить адресу в наступний "щабель" списку; після 3 підключень за короткий проміжок часу адреса потрапляє у `ssh_blacklist` на добу і блокується.
 
Аналогічно можна захистити WinBox (порт 8291) - дублюванням правил зі зміною `dst-port`.
 
Додатково - журналювання спроб входу:
```
/system logging
add topics=system,error,critical action=memory
add topics=account action=memory
```
 
---
 
## 2.4. Призначення адміністративних ролей
 
RouterOS використовує групи політик (`/user group`), аналог рівнів привілеїв Cisco:
 
```
/user group
add name=network-viewer policy=read,winbox comment="лише перегляд"
add name=network-operator policy=read,write,test,winbox comment="базові зміни, без sensitive/password"
add name=security-admin policy=local,ssh,winbox,read,write,policy,test,password,sensitive,api comment="повний доступ, окрім reboot/ftp"
add name=super-admin policy=local,telnet,ssh,ftp,reboot,read,write,policy,test,winbox,password,web,sniff,sensitive,api,romon comment="повний контроль"
```
 
Прив'язка користувачів:
```
/user
add name=viewer1 group=network-viewer password=<pass>
add name=noc1 group=network-operator password=<pass>
add name=secadmin group=security-admin password=<pass>
```
 
Політику `sensitive` (перегляд паролів/секретів у конфігурації) та `password` (зміна паролів) варто видавати лише обмеженому колу security-адміністраторів.
 
---
 
## 2.5. Деактивація застарілих/небезпечних методів підключення
 
```
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=yes
set api disabled=yes
set api-ssl disabled=yes
 
# Залишаємо лише SSH (з обмеженнями за адресою) та WinBox/WinBox через захищений тунель
set ssh disabled=no
set winbox disabled=no
 
/ip ssh
set strong-crypto=yes
 
# Вимкнення MAC-Telnet/MAC-WinBox для L2-discovery-доступу (небезпечно на транзитних портах)
/tool mac-server
set allowed-interface-list=none
/tool mac-server mac-winbox
set allowed-interface-list=none
 
# Вимкнення Neighbor Discovery (розкриває інформацію про пристрій в мережі)
/ip neighbor discovery-settings
set discover-interface-list=none
 
# Вимкнення Bandwidth Test Server, якщо не використовується
/tool bandwidth-server
set enabled=no
 
# Вимкнення застарілого/непотрібного графа трафіку та DNS-кешу для зовнішніх запитів
/ip dns
set allow-remote-requests=no
 
# UPnP — типове джерело неконтрольованого прокидання портів
/ip upnp
set enabled=no
 
# Вимкнення непотрібних пакетів (romon)
/tool romon
set enabled=no
```
 
---
 
# 3. Додаткові рекомендації (спільні для Cisco та MikroTik)
 
Ці пункти не входили до обов'язкового переліку, але суттєво підвищують загальний рівень захищеності пристрою:
 
1. **Точний час (NTP)** - критично для коректних міток часу в логах при розслідуванні інцидентів.
   - Cisco: `ntp server 192.168.21.95`
   - MikroTik: `/system ntp client set enabled=yes servers=192.168.21.95`
2. **Централізоване журналювання (syslog)** - навіть без AAA-сервера, окремий syslog-сервер (лише прийом логів) значно полегшує аудит.
   - Cisco: `logging host 10.0.0.20`
   - MikroTik: `/system logging action add name=remote target=remote remote=10.0.0.20`
3. **Резервне копіювання конфігурації** та контроль цілісності (порівняння з еталоном) - дозволяє швидко виявити несанкціоновані зміни
4. **Вимкнення невикористовуваних фізичних портів/інтерфейсів**, щоб унеможливити фізичне підключення сторонніх пристроїв
   - Cisco: `interface range Gi0/10-24` → `shutdown`
   - MikroTik: `/interface ethernet disable ether5`
5. **Мінімізація сервісів discovery-протоколів** (CDP/LLDP/MNDP) на портах, де немає потреби в автовиявленні сусідів - знижує розвідувальну цінність пристрою для зловмисника
6. **Регулярне оновлення прошивки/ОС** (Cisco IOS, RouterOS) для закриття відомих CVE
7. **Банер попередження про юридичну відповідальність** за несанкціонований доступ - на обох платформах (`banner login` у Cisco, `/ip service set ... banner` або MOTD у MikroTik через `/system note`)

---

> 📌 **Контрольний чек-лист впровадження:**
>
> - ✅ VTY/консоль обмежені ACL, увімкнено лише SSH, встановлено exec-timeout
> - ✅ Створено персональні облікові записи для кожного адміністратора, видалено/перейменовано дефолтні (`admin`)
> - ✅ Паролі хешовані (`secret`/`password` v3+), встановлена мінімальна довжина
> - ✅ Налаштовано `login block-for` (Cisco) / address-list брутфорс-захист (MikroTik)
> - ✅ Розподілено рольовий доступ (privilege levels/parser views — Cisco; user group policy — MikroTik)
> - ✅ Вимкнено Telnet, HTTP (незашифрований), FTP, small-services, застарілі discovery-протоколи де не потрібні
> - ✅ Заплановано регулярне резервне копіювання конфігурацій
> - ✅ Вимкнено невикористовувані фізичні порти
> - ✅ Налаштовано NTP та централізоване журналювання
> - ✅ Заплановано регулярне оновлення ПЗ пристроїв

---

!!! quote "Примітка"
    Стаття базується на офіційній документації Cisco TAC та адаптовано з використанням інструментів штучного інтелекту