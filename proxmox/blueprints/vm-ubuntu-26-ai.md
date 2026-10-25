# Ubuntu for AI services

## Ubuntu 26.04 Server LTS

1. Добавить ISO
1. Создать VM
    - **OS**: выбрать ISO, тип ОС
    - **System**:
        - Machine: `q35`
        - BIOS: `OVMF (UEFI)`
        - Pre-Enroll Keys: `❌` (если ее включить, то при установке драйверов видеокарты надо поработать с MOK Manager)
        - Qemu Agent: `✅`
    - **Disk**:  Добавить два диска, 32 ГБ (система; *Backup*: `✅`) и 64 ГБ (модели; *Backup*: `❌`):
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
    - **[Для работы с видеокартой]** перед запуском [пробросить видеокарту](../gpu-sharing.md#проброс-видеокарты-в-vm) как *PCI Device*
        - All Functions: `✅`
        - ROM-Bar: `✅`
        - PCI-Express: `✅`
1. Установить дистрибутив
    1. Базовый вариант установки, не *minimized*.
    1. Выбрать меньший диск для установки, *Set up this disk as an LVM group*: `❌`.
    1. *Install OpenSSH server*: `✅`.
1. Удалить ISO-образ Ubuntu из списка устройств, подключаться к VM по SSH.
1. Подключить второй диск:
    1. Найти диск командой `lsblk` (весьма вероятно, что **/dev/sdb**)
    1. Создать файловую систему
        ```bash
        sudo mkfs.ext4 /dev/sdb
        ```
    1. Создать папку для монтирования:
        ```bash
        sudo mkdir -p /mnt/storage
        sudo chown -R $USER:$USER /mnt/storage
        ```
    1. Настроить автоматическое монтирование диска:
        1. Узнать идентификатор UUID
            ```bash
            sudo blkid /dev/sdb
            ```
        1. В файле **/etc/fstab** добавить строку, подставив значение с прошлого шага
            ```ini
            UUID=ваш-скопированный-uuid-сюда /mnt/storage ext4 defaults 0 2
            ```
    1. Обновить файл конфигурации и подключить диск
        ```bash
        sudo systemctl daemon-reload
        sudo mount -a
        df -h /mnt/storage
        ```
1. Обновить дистрибутив
    - Проверить в файле **/etc/apt/sources.list.d/ubuntu.sources** строки с `Signed-by:` - они должны быть написаны в таком регистре; иначе возникнет ошибка.
    - Выполнить код:
        ```bash
        sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y
        sudo reboot
        ```

## SSH

1. В файле **/etc/ssh/sshd_config**
    - Сменить порт SSH: `Port новый_порт`.
    - Запретить вход под root: `PermitRootLogin no`
    - Разрешить вход по ключу: `PubkeyAuthentication yes`
1. Обновить конфигурацию SSH
    ```bash
    sudo systemctl restart ssh
    ```
1. **[Для Firewall]**: добавить правило `sudo ufw allow` с новым портом, можно с указанием ip с маской


### PubkeyAuth

1. В файле **/etc/ssh/sshd_config** запретить вход по логину: `PasswordAuthentication no`
1. Сгенерировать ключ на клиенте через *PuTTYgen*
    - **Windows**:
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
