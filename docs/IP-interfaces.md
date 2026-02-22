# Налштування ІР-адрес

Тут зібрані команди для налаштування IPv4 адресації

------------------------------------------------------------------------

# 1. Налаштування ІР-адреси та активація інтерфейсу
## Cisco IOS:
    ``` bash
    interface ethernet 0/0
    ip address 10.0.0.1 255.255.255.252
    no shutdown
    ```

## Mikrotik ROS:
    ``` bash
    /ip/address/add address=10.0.0.1/30 interface=ether1
    /interface/set ether1 disabled=no
    ```

## Linux:

    ```bash
    sudo ip address add 10.0.0.1/30 dev eth0
    sudo ip link set eth0 up
    ```

------------------------------------------------------------------------

# 2. Перегляд налаштованих адрес
## Cisco IOS:
    Повна інформація

    ``` bash
    show ip interface 
    ```
    Скорочена інформація

    ``` bash
    show ip interface brief
    ```
    Перегляд конфігурації конкретного інтерфейсу

    ``` bash
    show running-config interface ethernet 0/0
    ```

## Mikrotik ROS
    ``` bash
    /ip/address/print
    ```

## Linux
    ``` bash
    ip address
    ```

------------------------------------------------------------------------

# 3. Налаштування статичних маршрутів
## Cisco IOS:
Налаштування статичного маршруту


    ip route 172.16.0.0 255.240.0.0 10.0.0.2

Опціонально, може додаватись метрика (число від 1 до 255) для визначення пріоритету, чим менша метрика, тим більш пріоритетним ж маршрут
   

    ip route 0.0.0.0 0.0.0.0 10.0.0.2 7


Налаштування шлюза останньої надії


    ip default-gateway 10.0.0.1


## Mikrotik ROS:
