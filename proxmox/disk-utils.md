# Disk utils

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

## Mount disk

Проверялось только на Ubuntu.

1.**WEB-интерфейс**: Подключить *Hard drive* к VM

Далее по тексту - на клиенте.

1. Найти диск командой `lsblk` (вероятно, **/dev/sdb** - далее в примерах используется он)
1. Создать файловую систему:
    ```bash
    sudo mkfs.ext4 /dev/sdb
    ```

1. Создать папку для монтирования (для примера используется название *storage*):
    ```bash
    sudo mkdir -p /mnt/storage
    sudo chown -R $USER:$USER /mnt/storage
    ```

1. Настроить автоматическое монтирование диска:

    Узнать идентификатор UUID
    ```bash
    sudo blkid /dev/sdb
    ```

    В файле **/etc/fstab** добавить строку, подставив значение с прошлого шага
    ```ini
    UUID=ваш-скопированный-uuid-сюда /mnt/storage ext4 defaults 0 2
    ```

1. Обновить файл конфигурации и подключить диск
    ```bash
    sudo systemctl daemon-reload
    sudo mount -a
    df -h /mnt/storage
    ```

**Если диск удален**, VM запустится в аварийном режиме. Требуется удалить из автоматического монтирования добавленный диск из файла **/etc/fstab**.

## Share disk

Проверялось только на Ubuntu.

Это небезопасный способ расшарить один диск на несколько VM.

1. **Если новый диск**: Через WEB-интерфейс добавить диск как **Cache**: `No cache` на первую VM, запомнить его id (вида *scsiX*, где X - номер)
1. **Host shell**:
    1. В файле **/etc/pve/qemu-server/200.conf** (где 200 - id первой VM) в строке с *id* диска добавить параметр `shared=1`, скопировать всю строку в буфер обмена
    1. В файле **/etc/pve/qemu-server/300.conf** (где 300 - id второй VM) добавить скопированную строку, исправив номер диска, если он пересекается с другим
1. [Смонтировать диск](#mount-disk) в обеих VM

## Disk Stanby by timeout

`hdparm` — это классическая утилита командной строки для Linux, предназначенная для просмотра и настройки параметров HDD и SSD с интерфейсом ATA/SATA/SAS.

Для остановки шпинделя диска по таймауту можно выполнить команду `hdparm -S`. Таймер сбрасывается после перезагрузки.

### Скрипт остановки шпинделя по таймауту

1. Получить id дисков:
    ```bash
    ls -l /dev/disk/by-id/
    ```

1. Создать скрипт `/usr/local/bin/standby.sh` с содержимым:
    ```bash
    #!/bin/bash

    set -e

    DISKS=( # Перечислить свои диски
        "/dev/disk/by-id/ata-DISK1"
        "/dev/disk/by-id/ata-DISK2"
    )

    for disk in "${DISKS[@]}"; do
        if [[ -b "$disk" ]]; then
            hdparm -S 242 "$disk"
            logger -t hdd-spindown "Set 60 min spindown for $disk"
        else
            logger -t hdd-spindown "Disk not found: $disk"
        fi
    done
    ```

1. Сделать скрипт исполняемым:
    ```bash
    chmod +x /usr/local/sbin/hdd-spindown.sh
    ```

1. Создать systemd-сервис `/etc/systemd/system/hdd-spindown.service` с содержимым:
    ```ini
    [Unit]
    Description=Configure HDD spindown timers
    After=local-fs.target

    [Service]
    Type=oneshot
    ExecStart=/usr/local/sbin/hdd-spindown.sh
    RemainAfterExit=yes

    [Install]
    WantedBy=multi-user.target
    ```

1. Активировать сервис:
    ```bash
    systemctl daemon-reload
    systemctl enable hdd-spindown.service
    systemctl start hdd-spindown.service

    systemctl status hdd-spindown.service   # Проверить статус
    journalctl -u hdd-spindown.service      # Журнал сервиса
    ```
