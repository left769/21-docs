# 📚 Довідник кафедри інформаційних систем та технологій

Навчальний довідник-шпаргалка для студентів спеціальності **F6 Інформаційні системи та технології**.
Охоплює мережеві технології, мережеву безпеку та адміністрування Linux.

🔗 **[Відкрити довідник](https://ist.pp.ua/)**

---

## 📖 Зміст довідника

### 🌐 Мережі
| Розділ | Опис |
|--------|------|
| [Вступ до мереж](https://ist.pp.ua/network-configs/) | Базові концепції та моделі OSI/TCP-IP |
| [IP інтерфейси](https://ist.pp.ua/network-configs/IP-interfaces/) | Налаштування інтерфейсів, IPv4/IPv6 |
| [VLAN та Trunk](https://ist.pp.ua/network-configs/vlans/) | Сегментація мережі, 802.1Q |
| [VLAN в Mikrotik](https://ist.pp.ua/network-configs/vlans-mikrotik/) | Налаштування VLAN на маршрутизаторі Mikrotik |
| [STP](https://ist.pp.ua/network-configs/stp/) | Налаштування STP на Cisco |
| [VTP/DTP](https://ist.pp.ua/network-configs/vtp-dtp/) | Налаштування протоколів VTP та DTP |
| [Маршрутизація](https://ist.pp.ua/network-configs/routing/) | Статична та динамічна маршрутизація |
| [Dual WAN](https://ist.pp.ua/network-configs/isp-failover/) | Налаштування автоматичного переключення між провайдерами в Cisco |
| [Контроль доступу](https://ist.pp.ua/network-configs/ACL/) | ACL, фільтрація трафіку |
| [Cisco NAT](https://ist.pp.ua/network-configs/cisco-nat/) | Налаштування NAT на Cisco |
| [Cisco DHCP](https://ist.pp.ua/network-configs/dhcp/) | Налаштування DHCP-сервера на Cisco |
| [Cisco IPsec](https://ist.pp.ua/network-configs/ipsec/) | Налаштування IPsec-VPN між маршрутизаторами Cisco |
| [Mikrotik Wireguard](https://ist.pp.ua/network-configs/mikrotik-wireguard/) | Налаштування WireGuard на маршрутизаторі Mikrotik |
| [OSPF](https://ist.pp.ua/network-configs/ospf/) | Налаштування OSPFv2 на Cisco |
| [Захист пристроїв](https://ist.pp.ua/network-configs/basic-dev-sec/) | Базові заходи по захисту мережевих пристрїв|
| [Cisco IOS — Шпаргалка](https://ist.pp.ua/network-configs/cisco-cheat-sheet/) | Повний довідник команд Cisco IOS |
| [Калькулятор маски](https://ist.pp.ua/network-configs/subnet-calculator/) | Заcіб визначення параметрів ІР-мереж |

### 🖥️ Сервіси Linux
| Розділ | Опис |
|--------|------|
| [Сервіси на базі Linux](https://ist.pp.ua/server-services/) | Розгортання серверних сервісів |
| [IP addr](https://ist.pp.ua/server-services/linux-network-interfaces/) | Мережеві налаштування в Linux |
| [SSH](https://ist.pp.ua/server-services/ssh/) | Налаштування SSH |
| [RADIUS](https://ist.pp.ua/server-services/radius/) | Налаштування сервера автентифікації |
| [SYSLOG](https://ist.pp.ua/server-services/syslog/) | Налаштування syslog-сервера |
| [iptables](https://ist.pp.ua/server-services/iptables/) | Налаштування хостового фаєрволу |
| [TCPdump](https://ist.pp.ua/server-services/tcpdump-guide/) | Використання tcpdump |
| [chmod-калькулятор](https://ist.pp.ua/server-services/chmod-calculator/) | Калькулятор прав в Linux |

---

## 🚀 Локальний запуск

Щоб переглянути або редагувати довідник локально:
```bash
# 1. Клонувати репозиторій
git clone https://github.com/left769/21-docs.git
cd 21-docs

# 2. Встановити залежності
pip install mkdocs-material

# 3. Запустити локальний сервер
mkdocs serve
```

Після цього відкрий у браузері: `http://127.0.0.1:8000`

---

## ✏️ Як долучитися

Знайшов помилку або хочеш доповнити розділ? Будь ласка:

1. Зроби **Fork** цього репозиторію
2. Внеси зміни у відповідний `.md` файл у папці `docs/`
3. Створи **Pull Request** з коротким описом змін

Усі матеріали написані українською мовою.

---

## 🛠️ Технології

- [MkDocs](https://www.mkdocs.org/) — генератор статичних сайтів
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) — тема
- [GitHub Pages](https://pages.github.com/) — хостинг

Для додавання стилізованих іконок можна скористатися офіційним репозиторієм
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/reference/icons-emojis/)

---

## 👨🏻‍🔧 TO DO

Network:
```bash
- network-configs/bgp.md
- network-configs/mikrotik-cheat-sheet.md
- network-configs/mikrotik-ovpn.md
- network-configs/mikrotik-firewall.md
- network-configs/fortigate.md
```

Services:
```bash
- server-services/basics.md
- server-services/nfs.md
- server-services/bind9.md
- server-services/dhcp.md
- server-services/ldap.md
- server-services/git.md
- server-services/snort.md
- server-services/suricata.md
- server-services/nmap.md
- server-services/certbot.md
- server-services/netbox.md
- server-services/zabbix.md #Дані привіт
- server-services/ansible.md
- server-services/docker.md
- server-services/terraform.md
- server-services/haproxy.md
- server-services/openvpn.md
- server-services/wireguard.md
- server-services/email.md
- server-services/asterisk.md #Єгору привіт
```
---

## 📬 Контакти

**Кафедра інформаційних систем та технологій — Кафедра №21**

> Довідник постійно оновлюється. Якщо якийсь розділ порожній — він у розробці 🚧