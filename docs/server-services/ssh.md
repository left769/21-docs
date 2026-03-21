# SSH в Linux — Шпаргалка з налаштування

---

## 1 Встановлення та основи

### 1.1 Встановлення OpenSSH

```bash
apt update && apt install openssh-server -y

# Запуск та автозавантаження
systemctl start ssh
systemctl enable ssh
systemctl status ssh
```

### 1.2 Підключення до сервера

```bash
# Базове підключення
ssh user@192.168.1.10

# Вказати порт явно
ssh -p 2222 user@192.168.1.10

# Вказати приватний ключ
ssh -i ~/.ssh/id_rsa user@192.168.1.10

# Перевірити версію SSH
ssh -V
```

### 1.3 Основний конфігураційний файл

```bash
# Конфіг сервера
/etc/ssh/sshd_config

# Конфіг клієнта (для поточного користувача)
~/.ssh/config

# Після кожної зміни sshd_config — перезапустити сервіс
systemctl restart ssh

# Або перечитати конфіг без розриву поточних сесій
systemctl reload ssh
```

---

## 2 Аутентифікація за ключами

### 2.1 Генерація ключової пари

```bash
# RSA 4096 біт (сумісний варіант)
ssh-keygen -t rsa -b 4096 -C "user@hostname"

# Ed25519 (рекомендовано — коротший та надійніший)
ssh-keygen -t ed25519 -C "user@hostname"

# Ключ зберігається у:
~/.ssh/id_ed25519      # приватний ключ (нікому не передавати!)
~/.ssh/id_ed25519.pub  # публічний ключ (розміщується на сервері)
```

!!! warning "Захист приватного ключа"
    Приватний ключ `id_ed25519` — це аналог пароля. Ніколи не передавай його по мережі і не зберігай у хмарі без шифрування. При генерації обов'язково вказуй passphrase.

### 2.2 Розміщення публічного ключа на сервері

```bash
# Автоматичний спосіб (рекомендовано)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@192.168.1.10

# Ручний спосіб — скопіювати вміст .pub до файлу на сервері
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys

# Правильні права на файли (обов'язково!)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 2.3 Налаштування ~/.ssh/config на клієнті

```bash
# Зручний псевдонім для підключення
Host myserver
    HostName 192.168.1.10
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Після цього достатньо написати:
ssh myserver
```

---

## 3 Hardening — налаштування безпеки

Усі параметри редагуються у `/etc/ssh/sshd_config`

### 3.1 Зміна стандартного порту

```bash
# /etc/ssh/sshd_config
Port 2222                       # змінити з 22 на будь-який вільний (1024–65535)
```

!!! info "Навіщо змінювати порт?"
    Порт 22 постійно сканується ботами. Зміна порту не є захистом сама по собі, але суттєво зменшує кількість автоматичних атак у логах.

### 3.2 Заборона входу для root

```bash
# /etc/ssh/sshd_config
PermitRootLogin no                   # заборонити вхід під root повністю
# PermitRootLogin prohibit-password  # дозволити лише з ключем (компроміс)
```

### 3.3 Вхід тільки за ключами (без пароля)

```bash
# /etc/ssh/sshd_config
PasswordAuthentication no       # заборонити вхід за паролем
PubkeyAuthentication yes        # дозволити вхід за ключем
AuthorizedKeysFile .ssh/authorized_keys
```

!!! danger "Перед вимкненням паролів"
    Переконайся, що твій публічний ключ вже додано до `authorized_keys` і підключення за ключем працює. Інакше заблокуєш сам себе.

### 3.4 Обмеження доступу за користувачами та IP

```bash
# /etc/ssh/sshd_config

# Дозволити вхід тільки конкретним користувачам
AllowUsers admin devops

# Дозволити вхід тільки з конкретної підмережі
# (робиться через Match блок)
Match Address 192.168.1.0/24
    PasswordAuthentication yes

Match Address *
    PasswordAuthentication no
```

### 3.5 Додаткові параметри hardening

```bash
# /etc/ssh/sshd_config

Protocol 2                          # тільки SSH v2
X11Forwarding no                    # вимкнути forwarding графіки
AllowTcpForwarding no               # вимкнути TCP tunneling (якщо не потрібен)
MaxAuthTries 3                      # максимум 3 спроби аутентифікації
MaxSessions 5                       # максимум 5 одночасних сесій
LoginGraceTime 30                   # 30 секунд на аутентифікацію
ClientAliveInterval 300             # перевіряти активність клієнта кожні 5 хв
ClientAliveCountMax 2               # відключити якщо не відповідає 2 рази
Banner /etc/ssh/banner.txt          # показати банер перед входом
```

### 3.6 Перевірка конфігурації перед перезапуском

```bash
# Перевірити синтаксис sshd_config на помилки
sshd -t

# Детальна перевірка з виводом
sshd -T | grep -i "passwordauth\|permitroot\|port\|pubkey"
```

---

## 4 Управління ключами

### 4.1 Перегляд та видалення ключів

```bash
# Переглянути всі авторизовані ключі на сервері
cat ~/.ssh/authorized_keys

# Відкликати (видалити) конкретний ключ — відредагувати файл вручну
nano ~/.ssh/authorized_keys

# Переглянути відбиток (fingerprint) свого ключа
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# Переглянути відбиток ключа сервера
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

### 4.2 Відомі хости (known_hosts)

```bash
# Файл зберігає відбитки серверів до яких підключувався
~/.ssh/known_hosts

# Видалити запис про конкретний хост (після зміни ключа сервера)
ssh-keygen -R 192.168.1.10
ssh-keygen -R myserver

# Переглянути збережені хости
cat ~/.ssh/known_hosts
```

---

## 5 Корисні можливості SSH

### 5.1 Передача файлів

```bash
# SCP — копіювання файлів через SSH
scp file.txt user@192.168.1.10:/home/user/          # на сервер
scp user@192.168.1.10:/home/user/file.txt ./        # з сервера
scp -r ./folder user@192.168.1.10:/home/user/       # директорію рекурсивно
scp -P 2222 file.txt user@192.168.1.10:/tmp/        # вказати порт

# SFTP — інтерактивна передача файлів
sftp user@192.168.1.10
sftp> put file.txt          # завантажити файл
sftp> get file.txt          # скачати файл
sftp> ls                    # список файлів
sftp> exit
```

### 5.2 SSH тунелі

```bash
# Local forwarding — прокинути віддалений порт на локальний
# Доступ до RDP (3389) на 10.0.0.5 через SSH-сервер
ssh -L 3389:10.0.0.5:3389 user@192.168.1.10

# Remote forwarding — прокинути локальний порт на сервер
ssh -R 8080:localhost:80 user@192.168.1.10

# Dynamic forwarding — SOCKS5 proxy через SSH
ssh -D 1080 user@192.168.1.10
```

### 5.3 Виконання команд без інтерактивної сесії

```bash
# Виконати одну команду і відключитися
ssh user@192.168.1.10 "uptime"
ssh user@192.168.1.10 "df -h && free -m"

# Запустити скрипт на сервері
ssh user@192.168.1.10 'bash -s' < local_script.sh
```

---

## 6 Fail2ban — захист від brute-force

### 6.1 Встановлення

```bash
# Ubuntu/Debian
apt install fail2ban -y

# CentOS/RHEL
dnf install fail2ban -y

# Запуск
systemctl start fail2ban
systemctl enable fail2ban
```

### 6.2 Конфігурація

Fail2ban використовує два рівні конфігів:

- `/etc/fail2ban/jail.conf` — базовий (не редагувати!)
- `/etc/fail2ban/jail.local` — локальні перевизначення (редагувати тут)

```bash
# Створити локальний конфіг
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
nano /etc/fail2ban/jail.local
```

### 6.3 Налаштування jail для SSH

```ini
# /etc/fail2ban/jail.local

[DEFAULT]
bantime  = 1h                           # час блокування IP (1 година)
findtime = 10m                          # вікно для підрахунку невдалих спроб
maxretry = 5                            # максимум невдалих спроб перед баном
ignoreip = 127.0.0.1/8 192.168.1.0/24   # IP які ніколи не блокуються

[sshd]
enabled  = true
port     = 2222                         # вказати свій порт якщо змінено
logpath  = /var/log/auth.log
maxretry = 3                            # для SSH — суворіше ніж за замовчуванням
bantime  = 24h                          # банити на добу
```

```bash
# Перезапустити після змін
systemctl restart fail2ban
```

### 6.4 Управління та моніторинг

```bash
# Статус всіх jail
fail2ban-client status

# Статус конкретного jail
fail2ban-client status sshd

# Розблокувати IP вручну
fail2ban-client set sshd unbanip 192.168.1.10

# Заблокувати IP вручну
fail2ban-client set sshd banip 192.168.1.10

# Перевірити чи заблокований IP
fail2ban-client get sshd banip

# Переглянути логи fail2ban
tail -f /var/log/fail2ban.log
```

### 6.5 Перевірка роботи

```bash
# Переглянути заблоковані правила в iptables
iptables -L f2b-sshd -n -v

# Переглянути кількість спроб входу у логах
grep "Failed password" /var/log/auth.log | tail -20

# Переглянути успішні входи
grep "Accepted" /var/log/auth.log | tail -10
```

---

> 📌 **Обов'язковий чекліст після налаштування сервера:**
>
> - ✅ Змінено порт з 22 на нестандартний
> - ✅ `PermitRootLogin no`
> - ✅ `PasswordAuthentication no` (після додавання ключа!)
> - ✅ Fail2ban встановлено та запущено
> - ✅ Перевірено синтаксис: `sshd -t`
> - ✅ Перевірено підключення за ключем до перезапуску сервісу