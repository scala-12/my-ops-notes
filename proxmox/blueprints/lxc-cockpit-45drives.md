# LXC with Cockpit + 45Drives

*В примере будет использоваться **Debian 13**.*

## Preparing

1. Создать LXC с учетом рекомендаций из [записи](README.md#lxc-remarks):
    - **General**:
        - Hostname: cockpit45
        - Uprivileged container: ✅
        - Nesting: ✅
        - Template: *шаблон Debian 13*

    - **Disk**:
        - Storage: `Выбрать`
        - Disk size: `16` GiB

    - **CPU > Cores**: 2
    - **Memory**:
        - Memory: `2048` MiB
        - Swap: `512` MiB

    - **Network**: статичный адрес.

1. Запустить контейнер.

## Settings

1. Установить базовые пакеты:
    ```bash
    apt update && apt full-upgrade -y
    apt install -y \
        ca-certificates \
        curl \
        gnupg \
        apt-transport-https \
        sudo \
        dbus \
        systemd \
        rsync \
        zip \
        unzip \
        acl \
        samba \
        samba-common-bin
    ```

1. Установить `cockpit` и включить его:

    При установке будут ошибки - из-за не привилегированного контейнера. Не стоит обращать внимания, т.к. это связано с nfs-server, который в этой инструкции использоваться не будет.

    ```bash
    apt install -y cockpit cockpit-storaged
    systemctl enable --now cockpit.socket
    ```

    Проверить, что все прошло успешно:

    ```bash
    systemctl status cockpit.socket --no-pager
    ss -lntp | grep ':9090'
    ```

1. Подключить репозиторий `45Drives`:
    ```bash
    cd /root
    curl -fsSL https://repo.45drives.com/setup -o setup-45drives.sh

    less setup-45drives.sh          # Проверить скрипт; если все устараивает - установить
    bash /root/setup-45drives.sh
    apt update
    ```

    Проверить, что репозиторий появился и использует правильный код Debian:

    ```bash
    grep -Rni '45drives' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
    apt-cache policy cockpit-navigator cockpit-file-sharing cockpit-identities
    ```

    Для каждого пакета нужно увидеть доступную версию-кандидат (Candidate), а источником должен быть соответствующий репозиторий 45Drives. Если кандидат отсутствует, не переходи к установке вслепую — сначала нужно выяснить, какой пакет или источник недоступен.

1. Установить модули 45Drives:
    ```bash
    apt install -y \
        cockpit-navigator \
        cockpit-file-sharing \
        cockpit-identities

    systemctl restart cockpit.socket            # перезапуск сервиса Cockpit
    systemctl disable --now nfs-server.service  # отключить nfs
    ```

    Это устанавливает `файловый менеджер`, `управление ресурсами`, `управление пользователями` соответственно.

1. Создать группы пользователей:
    ```bash
    groupadd -g 2001 nas-ro
    groupadd -g 2002 nas-rw
    groupadd -g 2003 nas-admin
    ```

    - `nas-admin` - управление файлами через Cockpit.
    - `nas-ro` - чтение через Samba.
    - `nas-rw` - чтение и запись через Samba.

1. Создать пользователей:

    Для работы в Cockit:

    ```bash
    adduser cockpitadmin
    usermod -aG nas-admin,sudo cockpitadmin
    ```

1. Настроить конфиг Samba `/etc/samba/smb.conf`:

    1. В секции `[homes]` добавить строку `available = no` для скрытия домашней папки.
    1. В секции `[Global]` добавить строки для наследования ACL при работе через Samba:
        ```ini
        inherit acls = yes
        map acl inherit = yes
        ```

    1. Проверить и применить конфигурацию:
        ```bash
        testparm
        systemctl restart smbd
        ```

1. На хосте:

    Подключить необходимые каталоги в контейнер командой:
    ```bash
    pct set 120 -mp0 /mnt/storage,mp=/srv/storage
    ```

    *где `120` - номер контейнера, `-mp0` - номер точки монтирования, `/mnt/storage` - каталог на хосте, `/srv/storage` - каталог в контейнере.*

    Установить пакет `acl`:

    ```bash
    apt install acl -y
    ```

    Дать права Samba на чтение каталогов (readonly):

    ```bash
    setfacl -m \
        u:101000:rwx,g:102003:rwx,g:102001:r-x \
        /mnt/readonly

    setfacl -m \
        d:u::rwx,d:g::r-x,d:g:102003:rwx,d:g:102001:r-x,d:m::rwx,d:o::--- \
        /mnt/readonly
    ```

    Дать права Samba на чтение и запись:

    ```bash
    setfacl -m \
        u:101000:rwx,g:102003:rwx,g:102001:r-x,g:102002:rwx \
        /mnt/storage

    setfacl -m \
        d:u::rwx,d:g::r-x,d:g:102003:rwx,d:g:102001:r-x,d:g:102002:rwx,d:m::rwx,d:o::--- \
        /mnt/storage
    ```

    - `101000` - это UID `1000` пользователя *cockpitadmin*, если вы следовали инструкции. Иначе используйте свой UID.

1. Открыть Cockpit в браузере по ссылке `https://<ip>:9090`:

    - На вкладке *File Sharing* будет предупрежение об ошибке Samba - нажать *Исправить* (понадобится административный доступ).

        Добавить в **Share Configuration** папки для шаринга, выбрав *Read Only* там, где это требуется.

    - На вкладке *Identities* создать пользователя для работы с Samba, добавив ему группы `nas-ro` и `nas-rw`.
    - На вкладке *Navigator* возможно работать  только в режиме ограниченного доступа. В административном режиме доступ в файловой системе хоста не разрешен по данной инструкции.
