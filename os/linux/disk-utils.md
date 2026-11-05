# Disk utils

## Disk Stanby by timeout

`hdparm` — это классическая утилита командной строки для Linux, предназначенная для просмотра и настройки параметров HDD и SSD с интерфейсом ATA/SATA/SAS.

Для остановки шпинделя диска по таймауту можно выполнить команду `hdparm -S`. Таймер сбрасывается после перезагрузки.
Ниже представлена инструкция для создания и использования скрипта для остановки шпинделя.

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
