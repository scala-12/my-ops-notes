# Proxmox VE GPU sharing

## Проброс видеокарты в VM

1. Включить **IOMMU** в загрузчике для ЦП (**/etc/default/grub**)
    - **Intel**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"`
    - **AMD**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"`
1. Обновить загрузчик командой `update-grub`
1. Добавить модули vfio (**/etc/modules-load.d/vfio.conf**)
    ```
    vfio
    vfio_iommu_type1
    vfio_pci
    ```
1. Импортировать модули `update-initramfs -u -k all`
1. Запретить загрузку драйверов GPU (Nvidia и AMD) на хосте - дописать в **/etc/modprobe.d/pve-blacklist.conf**
    ```bash
    blacklist nouveau
    blacklist nvidia*
    blacklist amdgpu
    blacklist radeon
    ```
1. Отключить инициализацию видеокарты на хосте
    1. Найти ID GPU `lspci` вида `xx:xx`
    1. Получить из команды `lspci -n -s xx:xx` ID вида `xxxx:xxxx`
    1. Добавить в **/etc/modprobe.d/vfio.conf** `options vfio-pci ids=xxxx:xxxx,yyyy:yyyy disable_vga=1` с данными с прошлого шага
1. Перезагрузить, проверить работу IOMMU
    - **Intel**: вывод `dmesg | grep -e DMAR -e IOMMU` содержит *IOMMU enabled*
    - **AMD**: вывод `dmesg | grep -e DMAR -e IOMMU -e AMD-Vi` содержит *interrupt remapping*
1. В VM добавить PCI Device: видеокарту как Primary GPU, в расширенных настройках выбрать PCIe. После установить систему с пробросом клавиатуры и мыши
