# Proxmox VE utils

## Resize disk

Проверялось только на Ubuntu.

1. **WEB-интерфейс**:
    1. Выбрать диск для увеличения через интерфейс
    1. **Disk actions → Resize**: Добавить объем
1. Открыть консоль VM, найти диск командой `lsblk`
1. [**Если несколько разделов на диске**]: Увеличить раздел диска (в примере 2 для /dev/sda2)
    ```bash
    sudo growpart /dev/sda 2
    ```

1. Расширить том
    ```bash
    sudo resize2fs /dev/sda2
    ```
