# ZFS guide

## ZFS Pool creation

1. Создать пул ZFS с зеркальной конфигурацией:
    ```bash
    zpool create \
        -o ashift=12 \          # оптимизация под физические сектора 4K
        -O compression=lz4 \    # лёгкое сжатие
        -O atime=off \          # не обновлять время доступа к файлам
        -O xattr=sa \           # эффективное хранение расширенных атрибутов
        -O acltype=posixacl \   # поддержка POSIX ACL
        tank mirror \           # точка монтирования и тип пула
        /dev/disk/by-id/... /dev/disk/by-id/... # диски пула

    zpool status             # Проверить состояние пула
    zfs list                 # Проверить список пулов
    ```

    - Для создания dataset используется команда `zfs create tank/directory`
    - Для удаления пула используется команда `zpool destroy tank/directory`

## dataset

Создать dataset можно командой:
```bash
zfs create \
    -o mountpoint=/mnt/pve/directory \ # опционально: кастомная точка монтирования
    tank/directory  # название dataset

# проверка
zfs get all tank/directory
df -h /mnt/pve/directory        # Если указали точку монтирования
```

Если датасет был создан без точки монтирования, то ее можно добавить командой:

```bash
zfs set mountpoint=/mnt/directory tank/directory
```

Созданный dataset можно использовать как хранилище **Directory**:
1. В интерфейсе выбрать **Datacenter > Storage > Add > Directory**
1. Указать путь к *dataset*.
