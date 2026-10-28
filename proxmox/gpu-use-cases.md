# Proxmox VE GPU sharing

## GPU Passthrough

1. Включить IOMMU в загрузчике для ЦП файл **/etc/default/grub**
    - **Intel**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"`
    - **AMD**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"`

1. Обновить загрузчик
    ```bash
    update-grub
    ```

1. Добавить модули vfio в файле **/etc/modules-load.d/vfio.conf**
    ```apacheconf
    vfio
    vfio_iommu_type1
    vfio_pci
    ```

1. Импортировать модули
    ```bash
    update-initramfs -u -k all
    ```

1. Запретить загрузку драйверов GPU (Nvidia и AMD) на хосте в файле **/etc/modprobe.d/pve-blacklist.conf**
    ```apacheconf
    blacklist nouveau
    blacklist nvidia*
    blacklist amdgpu
    blacklist radeon
    ```

1. Отключить инициализацию видеокарты на хосте
    1. Найти ID GPU `lspci` вида `xx:xx`
    1. Скопировать из результата команды `lspci -n -s xx:xx` (ID GPU с предыдущего шага) ID вида `xxxx:xxxx`
    1. Добавить в **/etc/modprobe.d/vfio.conf** строку `options vfio-pci ids=xxxx:xxxx,yyyy:yyyy disable_vga=1`, подставив данные с предыдущего шага

1. Перезагрузить, проверить работу IOMMU
    - **Intel**: вывод `dmesg | grep -e DMAR -e IOMMU` содержит *IOMMU enabled*
    - **AMD**: вывод `dmesg | grep -e DMAR -e IOMMU -e AMD-Vi` содержит *interrupt remapping*

1. В VM добавить *PCI Device*, чаще всего как
    - All Functions: `✅`
    - ROM-Bar: `✅`
    - PCI-Express: `✅`

    *Primary GPU* используется для вывода изображения через это устройство (скорее всего, нужно отключить видеокарту в настройках VM).
