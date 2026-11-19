# LXC with Cockpit + 45Drives

*В примере будет использоваться **Debian 13**.*
1. [Подготовка](#preparing)
1. [Настройка](#settings)
    - [Подключить каталоги в контейнер](#mount-dirs)
    - [Настройка прав доступа](#set-acl)

---

## Preparing

1. Создать LXC с учетом рекомендаций из [записи](README.md#о-lxc):
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

1. При использовании Firewall открыть порты:
    ```bash
    sudo ufw allow from 192.168.1.0/24 to any port 445 proto tcp comment 'Доступ к файлам из локальной сети'
    sudo ufw deny 445/tcp comment 'Блокировка внешнего доступа к файлам'

    sudo ufw allow from 192.168.1.0/24 to any port 9090 proto tcp comment 'Доступ к консоли из локальной сети'
    sudo ufw deny 9090/tcp comment 'Блокировка внешнего к консоли'
    ```

1. [Настроить Samba](../disk-utils.md#samba-settings).
1. Открыть Cockpit в браузере по ссылке `https://<ip>:9090`:

    - На вкладке *File Sharing* будет предупрежение об ошибке Samba - нажать *Исправить* (понадобится административный доступ).

        Добавить в **Share Configuration** папки для шаринга, выбрав *Read Only* там, где это требуется.

    - На вкладке *Identities* создать пользователя для работы с Samba, добавив ему группы `nas-ro` и `nas-rw`.
    - На вкладке *Navigator* возможно работать  только в режиме ограниченного доступа. В административном режиме доступ в файловой системе хоста не разрешен по данной инструкции.
