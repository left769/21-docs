# Apt-Cacher-NG — посібник з розгортання та налаштування

> **Apt-Cacher-NG** — це проксі-сервер для apt пакетів. Замість того щоб кожна VM окремо качала пакети з інтернету — вони всі качають через один кеш. Перший запит іде в інтернет, всі наступні — з локального кешу

---

## 📋 Зміст

1. [Навіщо це потрібно](#навіщо-це-потрібно)
2. [Вимоги до сервера](#вимоги-до-сервера)
3. [Встановлення](#встановлення)
4. [Конфігурація](#конфігурація)
5. [Запуск та перевірка](#запуск-та-перевірка)
6. [Підключення клієнтів](#підключення-клієнтів)
7. [Fallback при недоступності](#fallback-при-недоступності)
8. [Статистика та моніторинг](#статистика-та-моніторинг)
9. [Корисні команди](#корисні-команди)
10. [Усунення неполадок](#усунення-неполадок)
11. [IaC розгортання](#iac-розгортання)

---

## Навіщо це потрібно

**Без Apt-Cacher-NG:**
```
VM-01 → інтернет → завантажує nginx (5 MB)
VM-02 → інтернет → завантажує nginx (5 MB)
VM-03 → інтернет → завантажує nginx (5 MB)
...
50 VM × 5 MB = 250 MB трафіку з інтернету
```

**З Apt-Cacher-NG:**
```
VM-01 → Apt-Cacher-NG → інтернет → завантажує nginx (5 MB) — кешує
VM-02 → Apt-Cacher-NG → віддає з кешу nginx (5 MB) — без інтернету
VM-03 → Apt-Cacher-NG → віддає з кешу nginx (5 MB) — без інтернету
...
Трафік з інтернету = 5 MB замість 250 MB
```

**Переваги:**
- 💾 **Економія трафіку** — пакети качаються з інтернету тільки один раз
- ⚡ **Швидкість** — локальна мережа швидша за інтернет
- 🛡️ **Надійність** — якщо інтернет впав але пакет є в кеші — VM все одно встановить
- 📊 **Статистика** — видно що і коли встановлювалось на всіх VM

---

## Вимоги до сервера

| Ресурс | Мінімум | Рекомендовано |
|--------|---------|---------------|
| CPU | 1 vCPU | 2 vCPU |
| RAM | 512 MB | 2 GB |
| Диск | 20 GB | 100 GB |
| ОС | Ubuntu 22.04+ | Ubuntu 24.04 LTS |

> 💡 Розмір диску залежить від кількості VM та пакетів. Для 10-50 VM рекомендується 100 GB.

---

## Встановлення

### Крок 1 — Оновлення системи

```bash
sudo apt update && sudo apt upgrade -y
```

### Крок 2 — Встановлення Apt-Cacher-NG

```bash
sudo apt install -y apt-cacher-ng
```

Під час встановлення з'явиться питання про налаштування — можна пропустити (натиснути Enter), ми налаштуємо вручну.

### Крок 3 — Перевірка встановлення

```bash
apt-cacher-ng --version
```

---

## Конфігурація

Основний конфіг знаходиться в `/etc/apt-cacher-ng/acng.conf`.

### Повна конфігурація

```bash
sudo nano /etc/apt-cacher-ng/acng.conf
```

Замінити вміст на:

```ini
# Директорія для кешу пакетів
CacheDir: /var/cache/apt-cacher-ng

# Директорія для логів
LogDir: /var/log/apt-cacher-ng

# Порт (стандартний)
Port: 3142

# Дозволяємо доступ з усієї мережі
BindAddress: 0.0.0.0

# Автоматичне очищення застарілих пакетів (днів)
ExTreshold: 4

# Підтримка HTTPS репозиторіїв
PassThroughPattern: .*

# Сторінка статистики
ReportPage: acng-report.html
```

### Створення директорій

```bash
sudo mkdir -p /var/cache/apt-cacher-ng
sudo mkdir -p /var/log/apt-cacher-ng
sudo chown apt-cacher-ng:apt-cacher-ng /var/cache/apt-cacher-ng
sudo chown apt-cacher-ng:apt-cacher-ng /var/log/apt-cacher-ng
```

---

## Запуск та перевірка

### Запуск сервісу

```bash
sudo systemctl enable apt-cacher-ng
sudo systemctl start apt-cacher-ng
```

### Перевірка статусу

```bash
sudo systemctl status apt-cacher-ng
```

Очікуваний вивід:
```
● apt-cacher-ng.service - Apt-Cacher NG software download proxy
     Active: active (running)
```

### Перевірка порту

```bash
ss -tlnp | grep 3142
```

### Перевірка через браузер

Відкрий в браузері:
```
http://<IP_СЕРВЕРА>:3142
```

Має відкритись сторінка **"Apt-Cacher NG Usage Information"** — сервіс працює.

---

## Підключення клієнтів

На **кожній VM** яка має використовувати Apt-Cacher-NG виконай:

### Крок 1 — Створити файл конфігурації

```bash
echo 'Acquire::http::Proxy "http://<IP_APT_CACHER>:3142/";
Acquire::https::Proxy "DIRECT";' | sudo tee /etc/apt/apt.conf.d/01proxy
```

Замінити `<IP_APT_CACHER>` на реальний IP сервера (в нашому випадку `192.168.21.85`):

```bash
echo 'Acquire::http::Proxy "http://192.168.21.85:3142/";
Acquire::https::Proxy "DIRECT";' | sudo tee /etc/apt/apt.conf.d/01proxy
```

### Крок 2 — Перевірити що проксі застосувався

```bash
apt-config dump | grep Proxy
```

Очікуваний вивід:
```
Acquire::http::Proxy "http://192.168.21.85:3142/";
Acquire::https::Proxy "DIRECT";
```

### Крок 3 — Перевірити що пакети йдуть через кеш

```bash
sudo apt update 2>&1 | head -5
```

В URL має бути `192.168.21.85:3142`:
```
Get:1 http://192.168.21.85:3142/archive.ubuntu.com/ubuntu noble InRelease
```

### Крок 4 — Встановити тестовий пакет

```bash
sudo apt install -y cowsay
cowsay "Apt-Cacher-NG працює!"
```

---

## Fallback при недоступності

Рядок `Acquire::https::Proxy "DIRECT"` забезпечує що:
- **HTTP** пакети → через Apt-Cacher-NG
- **HTTPS** пакети → напряму (Apt-Cacher-NG не підтримує HTTPS)

Для повного fallback якщо сервер недоступний — apt автоматично спробує підключитись напряму до репозиторіїв після таймауту.

### Перевірка fallback

```bash
# На сервері Apt-Cacher-NG — зупинити сервіс
sudo systemctl stop apt-cacher-ng

# На клієнті — спробувати встановити пакет
sudo apt install -y tree
# Пакет має встановитись напряму з репозиторію
```

```bash
# Повернути сервіс назад
sudo systemctl start apt-cacher-ng
```

---

## Статистика та моніторинг

### Веб-інтерфейс

Відкрий в браузері:
```
http://192.168.21.85:3142/acng-report.html
```

Показує:
- **Data fetched** — скільки завантажено з інтернету
- **Data served** — скільки віддано клієнтам з кешу
- **Cache efficiency** — hits/misses статистика

### Логи в реальному часі

```bash
sudo tail -f /var/log/apt-cacher-ng/apt-cacher.log
```

Формат логу:
```
timestamp|I|0|IP_клієнта|URL_пакету
```

- `I` — Import (завантажено з інтернету і закешовано)
- `O` — Output (віддано з кешу)

### Перевірка що конкретний пакет пройшов через кеш

```bash
sudo grep -i cowsay /var/log/apt-cacher-ng/apt-cacher.log
```

Приклад виводу:
```
1234567890|I|0|192.168.21.80|ua.archive.ubuntu.com/ubuntu/pool/universe/c/cowsay/cowsay_3.03.deb
1234567891|O|0|192.168.21.85|ua.archive.ubuntu.com/ubuntu/pool/universe/c/cowsay/cowsay_3.03.deb
```

### Розмір кешу

```bash
du -sh /var/cache/apt-cacher-ng/
```

---

## Корисні команди

| Команда | Опис |
|---------|------|
| `sudo systemctl start apt-cacher-ng` | Запустити сервіс |
| `sudo systemctl stop apt-cacher-ng` | Зупинити сервіс |
| `sudo systemctl restart apt-cacher-ng` | Перезапустити сервіс |
| `sudo systemctl status apt-cacher-ng` | Статус сервісу |
| `sudo apt-cacher-ng -c /etc/apt-cacher-ng ForeGround=1` | Запустити вручну для перевірки конфігу |
| `du -sh /var/cache/apt-cacher-ng/` | Розмір кешу |
| `sudo find /var/cache/apt-cacher-ng -name "*.deb" \| wc -l` | Кількість закешованих пакетів |

---

## Усунення неполадок

### ❌ Сервіс не стартує

```bash
sudo apt-cacher-ng -c /etc/apt-cacher-ng ForeGround=1
```

Покаже точну помилку в конфіги.

### ❌ Клієнт не бачить проксі

```bash
# Перевірити файл конфігу
cat /etc/apt/apt.conf.d/01proxy

# Перевірити що apt бачить проксі
apt-config dump | grep Proxy
```

### ❌ Пакети йдуть напряму, не через кеш

```bash
# Очистити кеш apt на клієнті
sudo apt clean
sudo apt update
```

### ❌ 503 Host not found на сторінці статистики

Перевірити що `ReportPage` в конфігу написаний на окремому рядку:

```bash
sudo grep ReportPage /etc/apt-cacher-ng/acng.conf
# Має показати: ReportPage: acng-report.html
# (тільки цей рядок, без злипання з іншими директивами)
```

### ❌ Port 3142 is busy

```bash
# Перевірити що процес вже запущений
ps aux | grep apt-cacher-ng
sudo systemctl status apt-cacher-ng
```

---
