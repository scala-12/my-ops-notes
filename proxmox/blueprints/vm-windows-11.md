# Установка Windows 11

1. Добавить ISO Windows 11 PRO (с RDP) и [дополнение ОС virtio-win](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso)
1. Создать VM
    - **OS**:
        - выбрать ISO
        - тип ОС: `Microsoft Windows`
        - Add additional drive for VirtIO drivers: `✅`
    - **System**: выбрать `EFI Storage`, `TPM Storage`
    - **Disk**:
        - Выбрать Storage
        - Disk size: `64GB`
        - Cache: `Write back`
        - Discard: `✅`
        - IO thread: `✅`
    - **CPU**:
        - Cores: `реальное число ядер`
        - Type: `host`
    - **Memory**: более 16ГБ
    - **Network**: Model `VirtIO (paravirtualized)`
    - **[Для работы через видеокарту]** перед запуском
        - [Пробросить видеокарту](../gpu-sharing.md#проброс-видеокарты-в-vm) как *PCI Device*
            - All Functions: `✅`
            - ROM-Bar: `✅`
            - PCI-Express: `✅`
        - Пробросить мышь и клавиатуру
        - **System**: Graphic card: `None`
1. Запустить VM, открыть консоль, установить Windows
    - На этапе выбора диска нажать "Загрузить" для установки драйверов.
    - После перезапуска для установки без сети: консоль `Shift + F10`, команда `oobe\bypassnro`
    - Установить дополнения гостевой ОС
