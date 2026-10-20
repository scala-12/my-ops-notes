# Установка Ubuntu 24.04 Server LTS

1. Добавить ISO

1. Создать VM
    - **OS**: выбрать ISO, тип ОС
    - **System**: 
        - Machine: `q35`,
        - BIOS: `OVMF (UEFI)`,
        - Pre-Enroll Keys: `❌`,
        - Qemu Agent: `✅`.
    - **Disk**:
        - Storage: `Выбрать`,
        - Disk size: `96GB`,
        - Cache: `Write back`,
        - Discard: `✅`,
        - IO thread: `✅`;
        - SSD emulation: `✅`,
        - Backup: `❌`.
    - **CPU**:
        - Cores: `реальное число ядер`,
        - Type: `host`.
    - **Memory**:
        - Memory: `больше - лучше`,
        - Balooning Device: `❌`.
    - **Network**: Model `VirtIO (paravirtualized)`
    - **Перед запуском** [пробросить видеокарту](../gpu-sharing.md#проброс-видеокарты-в-vm) как *PCI Device*:
        - All Functions: `✅`,
        - ROM-Bar: `✅`,
        - PCI-Express: `✅`.

1. [Установить дистрибутив](../../ai-services/install-os.md).

1. Удалить ISO-образ Ubuntu из списка устройств, подключаться к VM по SSH.
