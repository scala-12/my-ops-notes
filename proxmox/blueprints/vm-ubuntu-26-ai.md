# Ubuntu 26.04 Server LTS for AI services with GPU-passthrow

Не лучший вариант, потому что резервирует ОЗУ на хосте. Лучше использовать [LXC контейнер](./lxc-ubuntu-26-ai.md) с [шарингом видеокарты через хост](../gpu-use-cases.md#host-gpu-sharing).

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

        Под систему:
        - Storage: `Выбрать`
        - Disk size: `32 ГБ`
        - Cache: `No cache` (надежно и быстро)
        - Discard: `✅`
        - IO thread: `✅`
        - SSD emulation: `✅` (если SSD/NVMe)
        - Backup: `✅`

        Под модели выбрать такие же, как для основного диска, кроме:
        - Storage: `Выбрать`
        - Disk size: `120 ГБ`
        - Backup: `❌` (бекап моделей не нужен)

    - **CPU**:
        - Cores: `реальное число ядер`
        - Type: `host`

    - **Memory**:
        - Memory: `больше - лучше`
        - Balooning Device: `❌`

    - **Network**: Model `VirtIO (paravirtualized)`

    Перед запуском [пробросить](../gpu-sharing.md#gpu-passthrough) видеокарту как *PCI Device*:
    - All Functions: `✅`
    - ROM-Bar: `✅`
    - PCI-Express: `✅`

1. Установить дистрибутив
    1. Базовый вариант установки, не `minimized`.
    1. Выбрать диск для установки системы из ранее созданных: *Set up this disk as an LVM group*: `❌`.
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

1. [Настроить SSH](../../os/linux/ssh-settings.md)
