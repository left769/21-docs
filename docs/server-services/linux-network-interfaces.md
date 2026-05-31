# Linux мережеві інтерфейси - Шпаргалка

Комплексна інструкція для налаштування мережевих інтерфейсів в Linux за допомогою **ip**, **netplan** та інших інструментів

---

## Швидкий старт

### Показати всі мережеві інтерфейси
```bash
ip addr show              # Показати всі адреси
ip link show              # Показати стан всіх інтерфейсів
ip route show             # Показати таблицю маршрутизації
```

### Встановити IP адресу (тимчасово)
```bash
ip addr add 192.168.1.100/24 dev eth0     # Додати адресу
ip link set eth0 up                        # Включити інтерфейс
ip route add default via 192.168.1.1      # Додати маршрут по замовчуванню
```

### Встановити IP адресу (постійно через Netplan)
```bash
sudo nano /etc/netplan/01-netcfg.yaml
# Відредагувати файл (див. розділ Netplan)
sudo netplan apply
```

---

## IP Address - Керування IP адресами

### Перегляд адрес

| Команда | Опис |
|---------|------|
| `ip addr` | Показати всі IP адреси на всіх інтерфейсах |
| `ip addr show dev eth0` | Показати адреси тільки на інтерфейсі eth0 |
| `ip -4 addr show` | Показати тільки IPv4 адреси |
| `ip -6 addr show` | Показати тільки IPv6 адреси |

### Додавання адреси

```bash
# Базова адреса з CIDR маскою
ip addr add 192.168.1.100/24 dev eth0

# Додати вторинну адресу на той же інтерфейс
ip addr add 192.168.1.101/24 dev eth0

# Додати з лейблом
ip addr add 192.168.1.100/24 dev eth0 label eth0:1

# IPv6 адреса
ip addr add 2001:db8::1/64 dev eth0
```

### Видалення адреси

```bash
# Видалити адресу
ip addr del 192.168.1.100/24 dev eth0

# Видалити всі адреси з інтерфейсу (окрім loopback)
ip addr flush dev eth0
```

### Приклади

```bash
# Встановити адресу та включити інтерфейс
ip addr add 10.0.0.50/8 dev eth0
ip link set eth0 up

# Скинути всі адреси та встановити нові
ip addr flush dev eth0
ip addr add 192.168.10.10/24 dev eth0

# Додати кілька адрес
ip addr add 192.168.1.1/24 dev eth0
ip addr add 192.168.2.1/24 dev eth0
ip addr add 10.0.0.1/8 dev eth0
```

---

## IP Link - Керування інтерфейсами

### Перегляд інтерфейсів

| Команда | Опис |
|---------|------|
| `ip link` | Показати всі мережеві інтерфейси |
| `ip link show dev eth0` | Показати інформацію про інтерфейс eth0 |
| `ip -s link` | Показати статистику інтерфейсів |
| `ip link show type bond` | Показати тільки bond інтерфейси |

### Керування станом інтерфейсу

```bash
# Увімкнути інтерфейс
ip link set eth0 up

# Вимкнути інтерфейс
ip link set eth0 down

# Перевірити стан
ip link show dev eth0
```

### Зміна параметрів інтерфейсу

```bash
# Встановити MTU (Maximum Transmission Unit)
ip link set eth0 mtu 9000       # Jumbo frames
ip link set eth0 mtu 1500       # Стандартний розмір

# Увімкнути/вимкнути promiscuous режим (сніфінг)
ip link set eth0 promisc on
ip link set eth0 promisc off

# Змінити MAC адресу (інтерфейс повинен бути вимкнутий)
ip link set eth0 down
ip link set eth0 address 00:11:22:33:44:55
ip link set eth0 up

# Змінити назву інтерфейсу
ip link set eth0 name ens0
```

### Статистика

```bash
# Показати детальну статистику
ip -s link show dev eth0

# Одержати лише статистику помилок
ip -s link show dev eth0 | grep -E "RX|TX"
```

### Приклади

```bash
# Налаштування нового інтерфейсу
ip link set eth0 up
ip link set eth0 mtu 9000
ip addr add 192.168.1.10/24 dev eth0

# Перевірити статус
ip link show dev eth0
ip addr show dev eth0

# Вимкнути та включити інтерфейс (перезавантаження)
ip link set eth0 down
sleep 1
ip link set eth0 up
```

---

## IP Route - Керування маршрутизацією

### Перегляд маршрутів

| Команда | Опис |
|---------|------|
| `ip route` | Показати всі маршрути |
| `ip route show` | Альтернативна форма |
| `ip route show dev eth0` | Показати маршрути для інтерфейсу eth0 |
| `ip route get 8.8.8.8` | Показати, який маршрут буде використаний для адреси |

### Додавання маршрутів

#### Маршрут за замовчуванням
```bash
# Основний маршрут
ip route add default via 192.168.1.1 dev eth0

# З метрикою (вагою)
ip route add default via 192.168.1.1 dev eth0 metric 100
```

#### Статичний маршрут до мережі
```bash
# Через шлюз
ip route add 192.168.2.0/24 via 192.168.1.1

# Через інтерфейс
ip route add 192.168.2.0/24 dev eth0

# З метрикою
ip route add 192.168.2.0/24 via 192.168.1.1 metric 50
```

#### Маршрут до окремої машини
```bash
# До конкретної IP адреси
ip route add 10.0.0.50/32 via 192.168.1.1
```

### Видалення та заміна маршрутів

```bash
# Видалити маршрут
ip route delete 192.168.2.0/24 via 192.168.1.1

# Видалити маршрут по замовчуванню
ip route delete default

# Замінити маршрут (додати, якщо не існує)
ip route replace 192.168.2.0/24 via 192.168.1.1

# Очистити всі маршрути
ip route flush all
```

### Багатошляхова маршрутизація

```bash
# Додати два маршрути до однієї мережі через різні шлюзи
ip route add 192.168.3.0/24 via 192.168.1.1 metric 100
ip route add 192.168.3.0/24 via 192.168.2.1 metric 200
```

### Приклади

```bash
# Типове налаштування
ip route add default via 192.168.1.1 dev eth0

# Додати статичний маршрут та перевірити
ip route add 10.0.0.0/8 via 192.168.1.254
ip route show | grep 10.0.0.0

# Перевірити маршрут для конкретної адреси
ip route get 10.0.0.50

# Кілька маршрутів для забезпечення надійності
ip route add 8.8.8.8/32 via 192.168.1.1
ip route add 8.8.4.4/32 via 192.168.1.1
```

---

## Netplan - Постійне налаштування

**Netplan** - це утиліта для налаштування мережі на Ubuntu/Debian. Конфігурація зберігається в YAML файлах

### Розташування конфігурацій

```bash
/etc/netplan/         # Основна директорія
# Типові файли:
# - 00-installer-config.yaml
# - 01-netcfg.yaml
```

### Базові приклади

#### 1. Статична IP адреса

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

#### 2. DHCP (автоматичне отримання адреси)

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

#### 3. Кілька адрес на одному інтерфейсі

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
        - 192.168.1.101/24
        - 10.0.0.1/8
      gateway4: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 1.0.0.1]
```

#### 4. IPv6 адреса

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
        - 2001:db8::100/64
      gateway4: 192.168.1.1
      gateway6: 2001:db8::1
      nameservers:
        addresses: [8.8.8.8, 2001:4860:4860::8888]
```

#### 5. Статичні маршрути

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      routes:
        - to: 10.0.0.0/8
          via: 192.168.1.254
        - to: 172.16.0.0/12
          via: 192.168.1.253
          metric: 100
```

#### 6. Налаштування для двох інтерфейсів

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
    eth1:
      dhcp4: true
```

### Команди Netplan

```bash
# Перевірити синтаксис конфігурацій
sudo netplan validate

# Показати поточну конфігурацію
sudo netplan get

# Застосувати конфігурацію
sudo netplan apply

# Тимчасово застосувати для тестування (з можливістю відкату)
sudo netplan try

# Очистити та перезавантажити
sudo netplan apply --debug
```

### Практичні приклади

#### Повне налаштування з двома інтерфейсами

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    # Основний інтерфейс - статична адреса
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.50/24
      gateway4: 192.168.1.1
      nameservers:
        search: [example.com]
        addresses: [8.8.8.8, 8.8.4.4]
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
    
    # Другий інтерфейс - DHCP
    eth1:
      dhcp4: true
```

Застосування:
```bash
sudo netplan apply
sudo netplan query
```

---

## DHCP - Автоматичне отримання адреси

### Через Netplan

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true        # Увімкнути DHCP для IPv4
      dhcp6: true        # Увімкнути DHCP для IPv6
```

Застосування:
```bash
sudo netplan apply
```

### Через systemd-networkd

```ini
# /etc/systemd/network/20-dhcp.network
[Match]
Name=eth0

[Network]
DHCP=yes
```

Перезапуск:
```bash
sudo systemctl restart systemd-networkd
```

### Через dhclient (альтернативний варіант)

```bash
# Отримати адресу для інтерфейсу
sudo dhclient eth0

# Звільнити адресу (зупинити DHCP)
sudo dhclient -r eth0

# Обновити адресу
sudo dhclient -r eth0
sudo dhclient eth0
```

### Налаштування DHCP клієнта

```bash
# /etc/dhcp/dhclient.conf
# Встановити hostname
send host-name "myhost";

# Встановити domain
send domain-name "example.com";

# Встановити параметри для конкретного інтерфейсу
interface "eth0" {
  send host-name "web-server";
}
```

### Практичні приклади

#### Змішане налаштування (DHCP + статична маршрутизація)

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      routes:
        - to: 192.168.10.0/24
          via: 192.168.1.254
          metric: 100
      nameservers:
        addresses: [8.8.8.8]
```

#### Fallback налаштування (спочатку DHCP, потім статична)

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      addresses:
        - 192.168.1.100/24
```

---

## Bond - Агрегація інтерфейсів

Об'єднання кількох фізичних інтерфейсів в один логічний для підвищення пропускної спроможності та надійності.

### Через Netplan

#### 1. Active-Backup режим (основний + резервний)

```yaml
network:
  version: 2
  ethernets:
    eth0: {}
    eth1: {}
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: active-backup
        mii-monitor-interval: 100
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
```

#### 2. Balance-RR режим (розподіл навантаження)

```yaml
network:
  version: 2
  ethernets:
    eth0: {}
    eth1: {}
    eth2: {}
  bonds:
    bond0:
      interfaces: [eth0, eth1, eth2]
      parameters:
        mode: balance-rr
        mii-monitor-interval: 100
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
```

#### 3. Balance-ALB режим (адаптивний розподіл навантаження)

```yaml
network:
  version: 2
  ethernets:
    eth0: {}
    eth1: {}
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: balance-alb
        mii-monitor-interval: 100
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
```

#### 4. 802.3ad режим (LACP - потребує підтримки комутатора)

```yaml
network:
  version: 2
  ethernets:
    eth0:
      match:
        macaddress: "00:11:22:33:44:00"
    eth1:
      match:
        macaddress: "00:11:22:33:44:01"
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: 802.3ad
        lacp-rate: fast
        mii-monitor-interval: 100
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

### Команди для керування Bond'ом

```bash
# Показати інформацію про bond
cat /proc/net/bonding/bond0

# Показати статус bond'у через ip
ip link show bond0

# Показати адреси bond'у
ip addr show bond0

# Перевірити активний інтерфейс
cat /proc/net/bonding/bond0 | grep "Active Slave"
```

### Повна конфігурація з виключеннями

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    # Інтерфейси для bond'у
    eth0:
      dhcp4: no
    eth1:
      dhcp4: no
    
    # Окремий інтерфейс
    eth2:
      dhcp4: true
  
  bonds:
    # Основний bond для управління
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: 802.3ad
        lacp-rate: fast
        mii-monitor-interval: 100
      addresses:
        - 192.168.1.50/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
```

Застосування:
```bash
sudo netplan apply
```

### Переключення режимів Bond'у (тимчасово)

```bash
# За допомогою ip commands
sudo ip link set bond0 down
sudo ip link del bond0

# Створити новий bond з іншим режимом
echo "+bond0" > /sys/class/net/bonding_masters
echo "balance-alb" > /sys/class/net/bond0/bonding/mode
echo "+eth0" > /sys/class/net/bond0/bonding/slaves
echo "+eth1" > /sys/class/net/bond0/bonding/slaves
sudo ip link set bond0 up
sudo ip addr add 192.168.1.100/24 dev bond0
```

### Моніторинг Bond'у

```bash
# Безперервний моніторинг змін в bond'ові
watch cat /proc/net/bonding/bond0

# Перевірити статус інтерфейсів в bond'ові
grep -E "Slave Interface|MII Status" /proc/net/bonding/bond0
```

---

## Корисні команди

### Швидка діагностика

```bash
# Перевірити всі мережеві параметри
ip addr && echo "---" && ip link && echo "---" && ip route

# Перевірити підключення
ping -c 4 8.8.8.8

# Перевірити таблицю ARP
ip neigh show

# Перевірити DNS
nslookup example.com
dig example.com

# Перевірити послуги на портах
ss -tlnp
netstat -tlnp

# Перевірити мережеву статистику
ss -s
```

### Перегляд налаштувань інтерфейсу

```bash
# Детальна інформація про інтерфейс
ethtool eth0

# Драйвер та версія
ethtool -i eth0

# Швидкість та duplex
ethtool eth0 | grep Speed

# Статистика помилок
ethtool -S eth0 | grep -i error
```

### Пошук проблем

```bash
# Перевірити інтерфейси з помилками
for i in /sys/class/net/*/statistics/rx_errors; do
  echo "$i: $(cat $i)"
done

# Перевірити MTU всіх інтерфейсів
ip link | grep -E "^[0-9]:|mtu"

# Перевірити статус інтерфейсів real-time
watch -n 1 'ip -s link show'
```

### Резервне копіювання та відновлення конфігурації

```bash
# Резервна копія Netplan конфігурації
sudo cp -r /etc/netplan /etc/netplan.backup

# Експорт поточної конфігурації
sudo ip addr show > addr.backup
sudo ip route show > route.backup

# Відновлення з резервної копії
sudo cp /etc/netplan.backup/* /etc/netplan/
sudo netplan apply
```

### Переключення між конфігураціями

```bash
# Тестування конфігурації з автоматичним відкатом
sudo netplan try --timeout 120

# Якщо все добре, підтвердіть
# Якщо проблеми - чекайте 120 секунд для автоматичного відкату
```

---

## Порівняння старих та нових команд

| Стара команда | Нова команда (ip) |
|---------------|------------------|
| `ifconfig -a` | `ip addr show` |
| `ifconfig eth0 up` | `ip link set eth0 up` |
| `ifconfig eth0 down` | `ip link set eth0 down` |
| `ifconfig eth0 192.168.1.1/24` | `ip addr add 192.168.1.1/24 dev eth0` |
| `route -n` | `ip route show` |
| `route add default gw 192.168.1.1` | `ip route add default via 192.168.1.1` |
| `arp -a` | `ip neigh show` |
| `netstat` | `ss` |

---

## Швидкий довідник IP команд

### Синтаксис
```
ip [OPTION] OBJECT COMMAND
```

### Основні опції
| Опція | Опис |
|-------|------|
| `-4` | Показати тільки IPv4 |
| `-6` | Показати тільки IPv6 |
| `-s` | Статистика |
| `-r` | Роздільна назва (DNS) |

### Основні об'єкти (OBJECT)
| Об'єкт | Опис |
|--------|------|
| `addr` | IP адреси |
| `link` | Мережеві інтерфейси |
| `route` | Таблиця маршрутизації |
| `neigh` | ARP таблиця |
| `maddr` | Multicast адреси |

---

## Поради

1. **Тестування перед застосуванням**: Завжди використовуйте `netplan try` перед `netplan apply`
2. **Резервна копія**: Зберігайте оригінальну конфігурацію
3. **Моніторинг**: Перевіряйте логи (`journalctl -u systemd-networkd -f`)
4. **DNS**: Переконайтеся, що DNS налаштовано правильно
5. **Firewall**: Пам'ятайте про фаєрвол при налаштуванні маршрутів

---
!!! quote "Джерело"
    Стаття базується на офіційній документації Redhat
    Оригінал (англійською): [ip COMMAND CHEAT SHEET](https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf)
