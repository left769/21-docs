# Вітаю у довіднику!

Тут зібрані команди для налаштування обладнання.

## Базове налаштування IP
Виберіть вашу платформу:

=== "Cisco"
    ```ios
    conf t
    int eth0/0
    ip address 10.0.0.1 255.255.255.0
    no shut
    ```

=== "MikroTik"
    ```routeros
    /ip address add address=10.0.0.1/24 interface=ether1
    ```

=== "Linux"
    ```bash
    sudo ip addr add 10.0.0.1/24 dev eth0
    ```
