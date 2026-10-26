# Proxmox VE utils

## Resize disk

### Ubuntu VM

1. Выбрать диск для увеличения через интерфейс
1. **Disk actions → Resize**: Добавить объем
1. Открыть консоль машины, найти диск командой `lsblk`
1. [**Если несколько разделов**] Увеличить раздел диска (в примере 2 для /dev/sda2)
    ```bash
    sudo growpart /dev/sda 2
    ```
1. Расширить том
    ```bash
    sudo resize2fs /dev/sda2
    ```
