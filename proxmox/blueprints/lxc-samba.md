# LXC with SAMBA

*В примере будет использоваться **Debian 13**.*

## Preparing

1. Добавить Container template.
1. Создать LXC:
    - **General**:
        - Hostname: samba
        - Uprivileged container: ✅ (для безопасности)
        - Nesting: ✅ (Debian 13 ругается, если не включить)
        - Template: *шаблон Debian 13*

    - **Disk**:
        - Storage: `Выбрать`
        - Disk size: `8` GiB

    - **CPU > Cores**: 1
    - **Memory**:
        - Memory: `1024` MiB
        - Swap: `1024` MiB

    - **Network**: статичный адрес.

1. Подключить папки:
    ```bash
    pct set 100 -mp0 /mnt/share/software,mp=/srv/software,ro=1
    pct set 100 -mp1 /mnt/share/raw,mp=/srv/raw
    ```

1. Запустить контейнер.

## Settings

1. Установить пакеты:
    ```bash
    apt update
    apt install samba acl -y
    ```

1. Создать пользователя:
    ```bash
    useradd -M -s /usr/sbin/nologin -u 1000 storage
    smbpasswd -a storage
    ```

    На стандартном отображении UID непривилегированного LXC `UID 1000` внутри контейнера соответствует `UID 101000` на хосте.

1. На хосте настроить ACL:

    Установить пакет `acl`:

    ```bash
    apt install acl -y
    ```

    Дать права прохода через ACL:

    ```bash
    setfacl -m u:101000:--x /mnt # Добавить все родительские каталоги до требуемого каталога
    ```

    Дать права на каталог:

    ```bash
    setfacl -m u:101000:rx /mnt/share/software   # доступ к чтению
    setfacl -R -m u:101000:r-X /mnt/share/software

    setfacl -R -m u:101000:rwX /mnt/share/raw   # доступ к записи
    setfacl -R -d -m u:101000:rwX /mnt/share/raw
    ```

1. В контейнере настроить конфиг Samba `/etc/samba/smb.conf`:

    Добавить в существующие секции:

    ```ini
    [global]
        workgroup = WORKGROUP
        server string = Proxmox Samba
        security = user
        map to guest = never

    [homes]
        available = no
    ```

    Добавить в конце новые секции:

    ```ini
    [software]
        path = /srv/software
        browseable = yes
        read only = yes
        guest ok = no
        valid users = storage

    [raw]
        path = /srv/raw
        browseable = yes
        read only = no
        guest ok = no
        valid users = storage
        force user = storage
        create mask = 0660
        directory mask = 0770
    ```

    Проверить и применить конфигурацию:

    ```bash
    testparm
    systemctl restart smbd
    systemctl enable smbd
    ```
