# Proxmox VE GPU sharing

## GPU Passthrough

1. Добавить в загрузчик **/etc/default/grub** параметры для ЦП:
    - **Intel**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"`
    - **AMD**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"`

    Параметр `iommu=pt` (Pass-Through) переводит систему в режим, когда процессор не трогает устройства и не тратит ресурсы на их виртуализацию, пока его об этом явно не попросят.

    Параметр `intel_iommu=on` / `amd_iommu=on` активирует на уровне процессора технологию IOMMU (Intel VT-d / AMD-Vi).

1. Обновить загрузчик
    ```bash
    update-grub
    ```

1. Добавить модули vfio в файле **/etc/modules-load.d/vfio.conf**:
    ```apacheconf
    vfio
    vfio_iommu_type1
    vfio_pci
    ```

    *VFIO* - это специальный "виртуальный сейф". Когда устройство попадает в этот сейф, обычная операционная система теряет к нему доступ, а гипервизор KVM получает возможность передать это устройство внутрь виртуальной машины «как есть».

1. В файле **/etc/modprobe.d/pve-blacklist.conf** запретить на хосте загрузку драйверов GPU (Nvidia и AMD):
    ```apacheconf
    blacklist nouveau
    blacklist nvidia*
    blacklist amdgpu
    blacklist radeon
    ```

1. Импортировать модули
    ```bash
    update-initramfs -u -k all
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
