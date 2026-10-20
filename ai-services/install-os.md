# Установка дистрибутива Ubuntu 24.04 Server LTS

1. Базовый вариант установки, не *minimized*.

1. *Set up this disk as an LVM goup* `❌`.

1. *Install OpenSSH server* `✅`.

1. После установки обновить систему
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo reboot
    ```
