## Установка Windows 11

1. Добавить ISO Windows 11 PRO (с RDP) и [дополнение ОС](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso)
1. Создать VM
    - **OS**: выбрать ISO, тип ОС
    - **System**: выбрать EFI Storage, TPM Storage, Graphic card *None* (при пробросе GPU)
    - **Disk**: Выбрать Storage, Disk size *64GB*, Cache *Write back*, *Discard*, *IO thread*
    - **CPU**: Cores *реальное число ядер*, type *host*
    - **Memory**: более 16ГБ
    - **Network**: Model *VirtIO (paravirtualized)*
1. Запустить VM, открыть консоль, установить Windows
    - На этапе выбора диска нажать "Загрузить" для установки драйверов.
    - После перезапуска для установки без сети: консоль `Shift + F10`, команда `oobe\bypassnro`
    - Установить дополнения гостевой ОС