# Ubuntu 26.04 Server LTS for AI services with GPU-passthrow

## Preparing

1. Добавить ISO
1. Создать VM
    - **OS**: выбрать ISO, тип ОС
    - **System**:
        - Machine: `q35`
        - BIOS: `OVMF (UEFI)`
        - Pre-Enroll Keys: `❌` (если ее включить, то при установке драйверов видеокарты надо поработать с MOK Manager)
        - Qemu Agent: `✅`

    - **Disk**:
        - Storage: `Выбрать`
        - Disk size: `96GB`
        - Cache: `Write back`
        - Discard: `✅`
        - IO thread: `✅`
        - SSD emulation: `✅`

    - **CPU**:
        - Cores: `реальное число ядер`
        - Type: `host`

    - **Memory**:
        - Memory: `больше - лучше`
        - Balooning Device: `❌`

    - **Network**: Model `VirtIO (paravirtualized)`

    Перед запуском [пробросить](../gpu-sharing.md#проброс-видеокарты-в-vm) видеокарту как *PCI Device*:
    - All Functions: `✅`
    - ROM-Bar: `✅`
    - PCI-Express: `✅`

1. Установить дистрибутив
    1. Базовый вариант установки, не *minimized*.
    1. *Set up this disk as an LVM group*: `❌`.
    1. *Install OpenSSH server*: `✅`.

1. Удалить ISO-образ Ubuntu из списка устройств, подключаться к VM по SSH.

## Settings

### Main

1. Обновить дистрибутив
    - Проверить в файле **/etc/apt/sources.list.d/ubuntu.sources** строки с `Signed-by:` - они должны быть написаны в таком регистре; иначе возникнет ошибка.
    - Выполнить код:
        ```bash
        sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y
        sudo reboot
        ```

1. Настроить SSH

    В файле **/etc/ssh/sshd_config**
    - Сменить порт SSH: `Port новый_порт`.
    - Запретить вход под root: `PermitRootLogin no`
    - Разрешить вход по ключу: `PubkeyAuthentication yes`

    После чего обновить конфигурацию SSH
    ```bash
    sudo systemctl restart ssh
    ```

1. Добавить правило Firewall с новым портом SSD (можно с указанием ip с маской): `sudo ufw allow`

### PubkeyAuth

1. В файле **/etc/ssh/sshd_config** запретить вход по логину: `PasswordAuthentication no`
1. Сгенерировать ключ на клиенте через *PuTTYgen*

    **Windows**

    - *Тип ключа* `EdDSA Ed25519`
    - Generate
    - Сохранить поле *Public key for pasting* в файл для сервера
    - Сохранить ключи как **id_ed25519** с соответствующими расширениями

1. Скопировать *Public key for pasting* в **~/.ssh/authorized_keys** на сервере
1. Задать правильные права
    ```bash
    chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys && chown -R $USER:$USER ~/.ssh
    ```

1. Обновить конфигурацию SSH
    ```bash
    sudo systemctl restart ssh
    ```

1. **[Windows]**: Настроить использование ключей в PuTTY
    1. **Connection → SSH → Auth → Credentials**: выбрать ранее сгенерированный *Private key file*
    1. **Session**: Ввести адрес/порт и название, сохранить конфигурацию
    1. Подключиться
