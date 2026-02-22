# Розгортання та налаштування RADIUS сервера 

Тут зібрані наведено приклад розгортання ААА-сервера на основі пакету FreeRADIUS на ОС Linux Ubuntu 20.04

------------------------------------------------------------------------
## 1.   Розгортання FreeRADIUS 

1.1.    Оновіть систему та встановіть пакет FreeRADIUS

    apt update
    apt upgrade –y
    apt install –y freeradius freeradius-utils

1.2.    Запустіть сервіс та включіть автостарт

    systemctl start freeradius
    systemctl enable freeradius

1.3.    Згенеруйте сертифікат командами:

    cd /etc/freeradius/3.0/
    chmod +x bootstrap
    ./bootstrap

1.4.    Відкрийте в текстовому редакторі файл ```/etc/freeradius/3.0/users``` та додайте в кінець облікові дані користувача, котрим будете підключатись до мережевого обладнання:

    user Cleartext-Password := "P@$$w0rd"

1.5.    Відкрийте в текстовому редакторі файл ```/etc/freeradius/3.0/clients.conf``` та додайте в кінці параметри мережевих пристроїв котрим можна буде звертатись до вашого пристрою:

    client 10.60.254.1 {
    secret = C!sc0123
    }

1.6.    Перезавантажте сервіс FreeRADIUS командою:

    systemctl restart freeradius

------------------------------------------------------------------------

## 2.   Підключення мережевих пристроїв до ААА сервера

2.1    Cisco IOS

    aaa new-model
    aaa authentication login default group radius local
    aaa authorization exec default group radius local
    aaa accounting exec default start-stop group radius

    radius server rad1
      address ipv4 10.60.254.101 auth-port 1812 acct-port 1813
      key C!sc0123

    line vty 0 4
      transport input ssh