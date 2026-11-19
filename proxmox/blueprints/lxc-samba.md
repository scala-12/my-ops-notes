# LXC with SAMBA

*В примере будет использоваться **Debian 13**.*
1. [Подготовка](#preparing)
1. [Настройка](#settings)

---

## Preparing

1. Создать LXC с учетом рекомендаций из [записи](README.md#о-lxc):
    - **General**:
        - Hostname: samba
        - Uprivileged container: ✅
        - Nesting: ✅
        - Template: *шаблон Debian 13*

    - **Disk**:
        - Storage: `Выбрать`
        - Disk size: `8` GiB

    - **CPU > Cores**: 1
    - **Memory**:
        - Memory: `1024` MiB
        - Swap: `1024` MiB

    - **Network**: статичный адрес.

1. Запустить контейнер.

## Settings

1. Установить пакеты:
    ```bash
    apt update
    apt install samba acl -y
    ```

1. Создать пользователя Samba, добавить пароль и активировать его:
    ```bash
    useradd -M -s /usr/sbin/nologin -u 1000 storage
    smbpasswd -a storage
    smbpasswd -e storage
    ```

    На стандартном отображении UID непривилегированного LXC `UID 1000` внутри контейнера соответствует `UID 101000` на хосте.

1. [Настроить Samba](../disk-utils.md#samba-settings).
1. Добавить в секцию `[global]` параметр `guest ok = no`, а в конце файла добавить новые секции для папок Samba.
   К примеру:

    ```ini
    [Readonly]
        path = /srv/readonly
        browseable = yes
        read only = no
        valid users = @nas-ro @nas-rw

    [Write]
        path = /srv/write
        browseable = yes
        read only = no
        valid users = @nas-rw
    ```

    После проверить и применить конфигурацию:

    ```bash
    testparm
    systemctl restart smbd
    systemctl enable smbd
    ```
