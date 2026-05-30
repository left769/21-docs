# Rsyslog - мережевий сервер логів
 
> Курс: Безпека комп'ютерних мереж | 3 курс
 
---
 
## 1 Протокол Syslog
 
**Syslog** - стандартний протокол (RFC 5424) для передачі журналів подій між пристроями по мережі. Маршрутизатори, комутатори, сервери, міжмережеві екрани - всі вони можуть надсилати повідомлення на централізований **syslog-сервер** по UDP/514 або TCP/514
 
```
Cisco Router  ──┐
MikroTik      ──┤──► Rsyslog Server ──► SIEM / Архів
Linux Server  ──┘     (UDP/TCP 514)
```
 
**Rsyslog** - найпоширеніша реалізація syslog-сервера для Linux. Підтримує прийом подій по мережі, гнучку фільтрацію, збереження у файли, бази даних та пересилання на інші сервери
 
---
 
## 2 Структура Syslog-повідомлення
 
### 2.1 Приклад реального повідомлення з Cisco IOS
 
```
<189>Oct 15 08:23:11 R1 %SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: admin] [Source: 192.168.1.10] [localport: 22] at 08:23:11 UTC Mon Oct 15 2024
```
 
### 2.2 Розбір полів
 
```
<189>  Oct 15  08:23:11  R1  %SEC_LOGIN-5-LOGIN_SUCCESS: Login Success ...
  │       │        │      │         │
  │       │        │      │         └─ Message text
  │       │        │      └─ Hostname (source device)
  │       │        └─ Timestamp
  │       └─ Date
  └─ Priority = (Facility × 8) + Severity
```
 
| Поле | Значення | Опис |
|------|----------|------|
| `<189>` | Priority | PRI = Facility(23) × 8 + Severity(5) = 189 |
| `Oct 15 08:23:11` | Timestamp | Час події на пристрої |
| `R1` | Hostname | Ім'я пристрою-відправника |
| `%SEC_LOGIN-5-LOGIN_SUCCESS` | Mnemonic | Cisco-специфічний ідентифікатор: `%FACILITY-SEVERITY-MNEMONIC` |
| Текст після `:` | Message | Детальний опис події |
 
### 2.3 Facility - джерело повідомлення
 
| Код | Keyword | Призначення |
|-----|---------|-------------|
| 0 | `kern` | Повідомлення ядра Linux |
| 1 | `user` | Повідомлення від користувацьких процесів |
| 2 | `mail` | Поштова система |
| 3 | `daemon` | Системні демони |
| 4 | `auth` | Аутентифікація та безпека |
| 5 | `syslog` | Внутрішні повідомлення syslog |
| 6 | `lpr` | Принтери |
| 7 | `news` | Мережеві новини (NNTP) |
| 10 | `authpriv` | Приватні повідомлення аутентифікації |
| 16–23 | `local0`–`local7` | Призначені для користувача. **Cisco і MikroTik використовують `local7` за замовчуванням** |
 
### 2.4 Severity - рівень важливості
 
| Рівень | Keyword | Опис | Приклад |
|--------|---------|------|---------|
| 0 | `emerg` | Система непрацездатна | Критична помилка ядра |
| 1 | `alert` | Негайна дія | Зламаний RAID |
| 2 | `crit` | Критичний стан | Hardware failure |
| 3 | `err` | Помилки | Помилки інтерфейсу |
| 4 | `warning` | Попередження | Link down |
| 5 | `notice` | Важливі нормальні події | Вхід користувача |
| 6 | `info` | Інформаційні | DHCP lease |
| 7 | `debug` | Налагоджувальні | Детальний трейс |
 
!!! info "Як читати рівні severity"
    Правила типу `*.warning` означають **warning і вище по важливості** — тобто 0 (emerg), 1 (alert), 2 (crit), 3 (err), 4 (warning). Чим менший номер — тим критичніше
 
---
 
## 3 Встановлення та базова конфігурація Rsyslog
 
### 3.1 Встановлення
 
```bash
# Ubuntu/Debian
apt update && apt install rsyslog -y
 
# CentOS/RHEL
dnf install rsyslog -y
 
# Запуск та автозавантаження
systemctl start rsyslog
systemctl enable rsyslog
systemctl status rsyslog
```
 
### 3.2 Структура конфігурації
 
```
/etc/rsyslog.conf          ← головний конфіг (модулі, глобальні параметри)
/etc/rsyslog.d/            ← директорія для кастомних правил (*.conf)
/var/log/                  ← директорія для логів за замовчуванням
```
 
!!! tip "Кастомні правила - в /etc/rsyslog.d/"
    Не редагуй `/etc/rsyslog.conf` для додавання своїх правил. Натомість створюй окремі файли у `/etc/rsyslog.d/`. При оновленні пакету головний конфіг може перезаписатись, а файли в `rsyslog.d/` залишаться. Порядок завантаження файлів - алфавітний, тому іменуй їх з числовим префіксом: `10-network.conf`, `20-security.conf`
 
---
 
## 4 Активація мережевих модулів (прийом логів по мережі)
 
Відредагуй `/etc/rsyslog.conf` - розкоментуй або додай відповідні рядки:
 
```bash
# Завжди завантажуємо базові модулі
module(load="imuxsock")    # локальний /dev/log (системні логи)
module(load="imklog")      # логи ядра (dmesg)
 
# UDP - для Cisco, MikroTik та більшості мережевого обладнання
module(load="imudp")
input(type="imudp" port="514")
 
# TCP - для надійної доставки (Linux-сервери, rsyslog → rsyslog)
module(load="imtcp")
input(type="imtcp" port="514")
```
 
```bash
# Відкрити порти у firewall
ufw allow 514/udp
ufw allow 514/tcp
 
# або через firewalld
firewall-cmd --permanent --add-port=514/udp
firewall-cmd --permanent --add-port=514/tcp
firewall-cmd --reload
```
 
```bash
# Перевірити що сервер слухає на порту
ss -ulnp | grep 514
ss -tlnp | grep 514
```
 
---
 
## 5 Визначення директорій та файлів для збереження логів
 
### 5.1 Рекомендована структура директорій
 
```
/var/log/
├── remote/                      ← всі логи з мережевих пристроїв
│   ├── routers/
│   │   ├── cisco-r1.log
│   │   └── mikrotik-r2.log
│   ├── switches/
│   │   └── cisco-sw1.log
│   └── servers/
│       ├── web-server.log
│       └── db-server.log
├── security/                    ← події безпеки (окремо для SIEM)
│   ├── auth.log
│   ├── firewall.log
│   └── ids.log
└── syslog                       ← загальний журнал (стандартний)
```
 
```bash
# Створити директорії
mkdir -p /var/log/remote/{routers,switches,servers}
mkdir -p /var/log/security
 
# Права доступу
chown -R syslog:adm /var/log/remote /var/log/security
chmod -R 750 /var/log/remote /var/log/security
```
 
### 5.2 Налаштування шаблону імені файлу (динамічні файли)
 
Додай у `/etc/rsyslog.conf` або в окремий файл:
 
```
# Шаблон: зберігати лог кожного хоста у свій файл
# /var/log/remote/routers/192.168.1.1.log або /var/log/remote/routers/R1.log
template(name="RemoteHostLog" type="string"
         string="/var/log/remote/%HOSTNAME%.log")
 
template(name="RemoteIPLog" type="string"
         string="/var/log/remote/%FROMHOST-IP%.log")
 
# Шаблон з датою (зручно для ротації)
template(name="DailyRemoteLog" type="string"
         string="/var/log/remote/%HOSTNAME%/%$YEAR%-%$MONTH%-%$DAY%.log")
```
 
---
 
## 6 Кастомні правила фільтрації та збереження
 
Всі кастомні правила рекомендується розміщувати в `/etc/rsyslog.d/`
 
### 6.1 Фільтрація за IP-адресою
 
Файл: `/etc/rsyslog.d/10-by-ip.conf`
 
```
# Логи з конкретного IP - в окремий файл
if $FROMHOST-IP == '192.168.1.1' then /var/log/remote/routers/cisco-r1.log
if $FROMHOST-IP == '192.168.1.2' then /var/log/remote/routers/mikrotik-r2.log
if $FROMHOST-IP == '192.168.2.1' then /var/log/remote/switches/cisco-sw1.log
 
# Логи з підмережі маршрутизаторів - в загальний файл роутерів
if $FROMHOST-IP startswith '192.168.1.' then /var/log/remote/routers/all-routers.log
 
# Зупинити обробку після запису (& stop - не дублювати в інші правила)
if $FROMHOST-IP == '192.168.1.1' then {
    /var/log/remote/routers/cisco-r1.log
    stop
}
```
 
### 6.2 Фільтрація за hostname
 
Файл: `/etc/rsyslog.d/11-by-hostname.conf`
 
```
# За точним іменем хоста
if $HOSTNAME == 'R1' then /var/log/remote/routers/cisco-r1.log
if $HOSTNAME == 'MikroTik' then /var/log/remote/routers/mikrotik.log
 
# За шаблоном імені (всі хости що починаються на SW-)
if $HOSTNAME startswith 'SW-' then /var/log/remote/switches/all-switches.log
if $HOSTNAME startswith 'SRV-' then /var/log/remote/servers/all-servers.log
```
 
### 6.3 Фільтрація за facility та severity (тип повідомлення)
 
Файл: `/etc/rsyslog.d/20-by-type.conf`
 
```
# Всі повідомлення аутентифікації (auth та authpriv)
auth,authpriv.*                 /var/log/security/auth.log
 
# Повідомлення від local7 (Cisco/MikroTik за замовчуванням)
local7.*                        /var/log/remote/network-devices.log
 
# Тільки критичні та вище — з будь-якого джерела
*.crit                          /var/log/security/critical.log
 
# Повідомлення рівня warning і вище з мережевих пристроїв
if ($syslogfacility-text == 'local7') and ($syslogseverity <= 4) then {
    /var/log/security/network-warnings.log
}
 
# Ігнорувати debug-повідомлення (зекономити місце)
*.debug                         stop
```
 
### 6.4 Фільтрація за вмістом повідомлення (regex)
 
Файл: `/etc/rsyslog.d/30-security-events.conf`
 
```
# Повідомлення про вхід/вихід користувачів (Cisco LOGIN/LOGOUT)
if $msg contains 'LOGIN_SUCCESS' or $msg contains 'LOGIN_FAILED' then {
    /var/log/security/logins.log
}
 
# Cisco ACL deny — блокований трафік
if $msg contains '%SEC-6-IPACCESSLOGP' or $msg contains '%SEC-6-IPACCESSLOGS' then {
    /var/log/security/firewall.log
}
 
# Interface down/up події
if $msg contains 'changed state to down' or $msg contains 'changed state to up' then {
    /var/log/remote/interface-events.log
}
 
# Помилки конфігурації Cisco
if $msg contains '%SYS-5-CONFIG_I' then {
    /var/log/security/config-changes.log
}
```
 
### 6.5 Комплексне правило з кількома умовами
 
Файл: `/etc/rsyslog.d/40-complex.conf`
 
```
# Всі події безпеки з мережевих пристроїв (local7, severity 0-5) — у security-лог
if ($syslogfacility-text == 'local7') and \
   ($syslogseverity <= 5) and \
   ($FROMHOST-IP startswith '192.168.') \
then {
    action(type="omfile" file="/var/log/security/network-security.log"
           template="RSYSLOG_TraditionalFileFormat")
}
 
# Зберегти в файл за хостом І у загальний security-файл (без stop — обидва спрацюють)
if $FROMHOST-IP == '192.168.1.1' then /var/log/remote/routers/cisco-r1.log
if $syslogfacility-text == 'local7' and $syslogseverity <= 4
    then /var/log/security/network-warnings.log
```
 
---
 
## 7 Пересилання логів на інший сервер (relay / SIEM)
 
Типова ситуація: локальний rsyslog зберігає копію подій у файл **і одночасно пересилає їх** на SIEM або центральний лог-сервер відділу кібербезпеки
 
```
[Cisco/MikroTik] ──UDP──► [Rsyslog Server] ──TCP──► [SIEM / Central Syslog]
                                  │
                                  └──► /var/log/remote/ (локальна копія)
```
 
Файл: `/etc/rsyslog.d/99-forward.conf`
 
```
# Варіант 1 - пересилати ВСЕ на SIEM по TCP (надійно)
*.* action(type="omfwd"
           target="10.0.0.100"
           port="514"
           protocol="tcp"
           action.resumeRetryCount="100"
           queue.type="linkedList"
           queue.size="10000")
 
# Варіант 2 - пересилати тільки security-події на SIEM
if ($syslogfacility-text == 'local7') or \
   ($syslogfacility-text == 'auth') or \
   ($syslogfacility-text == 'authpriv') \
then {
    action(type="omfwd"
           target="10.0.0.100"
           port="6514"
           protocol="tcp"
           action.resumeRetryCount="50"
           queue.type="linkedList"
           queue.size="5000"
           queue.filename="siem-queue"
           queue.saveOnShutdown="on")
}
 
# Варіант 3 - зберегти локально І переслати (без stop між правилами)
auth,authpriv.*   /var/log/security/auth.log
auth,authpriv.*   action(type="omfwd" target="10.0.0.100" port="514" protocol="tcp")
```
 
!!! info "Черга (queue) при пересиланні"
    `queue.type="linkedList"` і `queue.size="10000"` - буфер повідомлень на випадок недоступності SIEM. Якщо SIEM тимчасово не відповідає, rsyslog буде зберігати події в пам'яті (або на диску з `queue.filename`) і надішле їх коли з'єднання відновиться. Без черги події при недоступності SIEM втрачаються
 
!!! tip "queue.saveOnShutdown"
    `queue.saveOnShutdown="on"` - зберігає чергу на диск при зупинці rsyslog. При наступному запуску черга відновлюється і всі буферизовані події надсилаються. Необхідно вказати `queue.filename` для цього режиму
 
---
 
## 8 Налаштування logrotate
 
Без ротації лог-файли будуть рости необмежено і заповнять диск
 
```bash
# Перевірити наявний конфіг (зазвичай вже є)
cat /etc/logrotate.d/rsyslog
```
 
Файл: `/etc/logrotate.d/remote-logs`
 
```
/var/log/remote/**/*.log
/var/log/security/*.log
{
    daily                    # ротувати щодня
    rotate 90                # зберігати 90 файлів (3 місяці)
    compress                 # стиснути старі файли (gzip)
    delaycompress            # стискати не відразу, а при наступній ротації
    missingok                # не повідомляти про помилку якщо файл відсутній
    notifempty               # не ротувати порожні файли
    create 0640 syslog adm  # права на новий файл
    sharedscripts            # виконати postrotate один раз для всіх файлів
    postrotate
        /usr/lib/rsyslog/rsyslog-rotate   # надіслати HUP сигнал rsyslog
    endscript
}
```
 
```bash
# Перевірити конфігурацію logrotate (без реального виконання)
logrotate -d /etc/logrotate.d/remote-logs
 
# Примусова ротація (для тесту)
logrotate -f /etc/logrotate.d/remote-logs
 
# Перевірити стан ротації
cat /var/lib/logrotate/status | grep remote
```
 
---
 
## 9 Налаштування remote logging на мережевому обладнанні
 
### 9.1 Cisco IOS
 
```
! Вказати адресу syslog-сервера
logging host 192.168.100.10
 
! Або з вказанням протоколу та порту (IOS 12.4+)
logging host 192.168.100.10 transport tcp port 514
logging host 192.168.100.10 transport udp port 514
 
! Рівень повідомлень що надсилаються (informational = 0-6)
logging trap informational
 
! Facility (local7 за замовчуванням, можна змінити)
logging facility local7
 
! Включити timestamp у повідомленнях
service timestamps log datetime msec localtime show-timezone
 
! Включити source-interface (щоб лог завжди йшов з одного IP)
logging source-interface Loopback0
 
! Увімкнути буфер локальних логів
logging buffered 16384 informational
 
! Перевірка
show logging
```
 
### 9.2 MikroTik RouterOS
 
```
# Додати remote syslog action
/system logging action
add name=remote-syslog \
    target=remote \
    remote=192.168.100.10 \
    remote-port=514 \
    src-address=0.0.0.0 \
    bsd-syslog=yes \
    syslog-facility=local7 \
    syslog-severity=auto
 
# Призначити всі топіки до remote action
/system logging
add topics=info    action=remote-syslog
add topics=warning action=remote-syslog
add topics=error   action=remote-syslog
add topics=critical action=remote-syslog
add topics=firewall action=remote-syslog
 
# Перевірка
/system logging print
/system logging action print
/log print
```
 
---
 
## 10 Debug та діагностика
 
### 10.1 Перевірка rsyslog
 
```bash
# Статус сервісу
systemctl status rsyslog
 
# Перевірити конфігурацію на помилки (обов'язково після кожної зміни!)
rsyslogd -N1              # перевірка синтаксису
rsyslogd -N2              # більш детальна перевірка
 
# Переглянути власні логи rsyslog
journalctl -u rsyslog -f
tail -f /var/log/syslog | grep rsyslog
 
# Перезапустити після змін конфігурації
systemctl restart rsyslog
# або м'який перезапуск (без втрати поточних з'єднань)
systemctl reload rsyslog
```
 
### 10.2 Перевірка мережевого прийому
 
```bash
# Слухати вхідні UDP syslog-пакети (до запуску rsyslog або для діагностики)
tcpdump -i any udp port 514 -nn -A
 
# Перевірити що rsyslog слухає на порту
ss -ulnp | grep 514
ss -tlnp | grep 514
 
# Надіслати тестове повідомлення з поточного хоста
logger -p local7.info "Test message from $(hostname)"
logger -p auth.warning "Test auth warning"
 
# Надіслати тестове повідомлення на конкретний syslog-сервер (netcat)
echo "<134>Oct 15 08:00:00 testhost TEST: hello syslog" | nc -u 192.168.100.10 514
```
 
### 10.3 Перевірка прийому логів з конкретного пристрою
 
```bash
# Стежити за надходженням логів у реальному часі
tail -f /var/log/remote/routers/cisco-r1.log
 
# Фільтрувати за IP-адресою в загальному syslog
tail -f /var/log/syslog | grep '192.168.1.1'
 
# Переглянути кількість повідомлень від кожного хоста за останню годину
grep "$(date '+%b %e %H')" /var/log/syslog | awk '{print $4}' | sort | uniq -c | sort -rn
 
# Переглянути розмір лог-файлів
du -sh /var/log/remote/**
ls -lh /var/log/remote/routers/
```
 
### 10.4 Типові проблеми
 
| Симптом | Причина | Рішення |
|---------|---------|---------|
| Логи не надходять | Порт 514 закритий | `ufw allow 514/udp` |
| Файл не створюється | Немає директорії або прав | `mkdir -p /var/log/remote && chown syslog:adm /var/log/remote` |
| Синтаксична помилка | Помилка у конфізі | `rsyslogd -N1` |
| Дублювання записів | Правило без `stop` | Додати `stop` після запису у файл |
| Логи йдуть у `/var/log/syslog` замість потрібного файлу | Правило не спрацьовує | Перевірити синтаксис умови та назву facility |
 
---
 
> 📌 **Після кожної зміни конфігурації:**
> ```bash
> rsyslogd -N1 && systemctl restart rsyslog
> ```
 
---
 
!!! quote "Джерела"
    [Rsyslog Documentation](https://docs.rsyslog.com/doc/configuration/index.html) ·
    [RFC 5424 — The Syslog Protocol](https://datatracker.ietf.org/doc/html/rfc5424) ·
    [Cisco IOS Logging](https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/system-monitoring/24xx/configuration/guide/b-system-monitoring-cg-cisco8k-24xx/implementing-system-logging.html)