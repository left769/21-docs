# 📚 Довідник кафедри інформаційних систем та технологій

Навчальний довідник-шпаргалка для студентів спеціальності **F6 Інформаційні системи та технології**.
Охоплює мережеві технології, мережеву безпеку та адміністрування Linux.

🔗 **[Відкрити довідник](https://left769.github.io/21-docs/)**

---

## 📖 Зміст довідника

### 🌐 Мережі
| Розділ | Опис |
|--------|------|
| [Вступ до мереж](https://left769.github.io/21-docs/network-configs/) | Базові концепції та моделі OSI/TCP-IP |
| [IP інтерфейси](https://left769.github.io/21-docs/network-configs/ip-interfaces/) | Налаштування інтерфейсів, IPv4/IPv6 |
| [VLAN та Trunk](https://left769.github.io/21-docs/network-configs/vlans/) | Сегментація мережі, 802.1Q |
| [Маршрутизація](https://left769.github.io/21-docs/network-configs/routing/) | Статична та динамічна маршрутизація |
| [Контроль доступу](https://left769.github.io/21-docs/network-configs/acl/) | ACL, фільтрація трафіку |
| [Cisco IOS — Шпаргалка](https://left769.github.io/21-docs/network-configs/cisco-cheat-sheet/) | Повний довідник команд Cisco IOS |

### 🖥️ Сервіси Linux
| Розділ | Опис |
|--------|------|
| [Сервіси на базі Linux](https://left769.github.io/21-docs/server-services/) | Розгортання серверних сервісів |
| [SSH](https://left769.github.io/21-docs/server-services/ssh/) | Налаштування SSH |
| [RADIUS](https://left769.github.io/server-services/radius/) | Налаштування сервера автентифікації |

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

---

## 👨🏻‍🔧 TO DO

Network:
```bash
- network-configs/stp.md
- network-configs/ipsec.md
- network-configs/ospf.md
- network-configs/bgp.md
- network-configs/mikrotik-cheat-sheet.md
- network-configs/mikrotik-ovpn.md
- network-configs/mikrotik-wireguard.md
- network-configs/microtik-firewall.md
- network-configs/dhcp.md
- network-configs/cisco-nat.md
- network-configs/fortigate.md
```

Services:
```bash
- server-services/basics.md
- server-services/nfs.md
- server-services/bind9.md
- server-services/dhcp.md
- server-services/syslog.md
- server-services/ldap.md
- server-services/ssh.md
- server-services/git.md
- server-services/snort.md
- server-services/suricata.md
- server-services/iptables.md
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