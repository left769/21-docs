# Налштування ІР-адрес

Тут зібрані команди для налаштування IPv4 адресації

------------------------------------------------------------------------

## 1. Налаштування ІР-адреси та активація інтерфейсу
Cisco IOS:

    interface ethernet 0/0
    ip address 10.0.0.1 255.255.255.252
    no shutdown

Mikrotik ROS:

    /ip/address/add address=10.0.0.1/30 interface=ether1
    /interface/set ether1 disabled=no

Linux:

    sudo ip address add 10.0.0.1/30 dev eth0
    sudo ip link set eth0 up

------------------------------------------------------------------------

## 2. Перегляд налаштованих адрес
Cisco IOS:
Повна інформація

    show ip interface 

Скорочена інформація

    show ip interface brief

Перегляд конфігурації конкретного інтерфейсу

    show running-config interface ethernet 0/0

Mikrotik ROS

    /ip/address/print

Linux

    ip address

------------------------------------------------------------------------
