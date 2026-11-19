# Disk utils

1. [Изменение размера диска](#resize-disk)
1. [Монтирование диска](#mount-disk)
1. [Шаринг диска между VM](#share-disk)
1. [Подключение ZFS датасета как Directory](#using-zfs-dataset-as-directory)
1. [Настройка общего доступа к файлам для Samba](#samba-settings)

---

## Resize disk

Проверялось только на Ubuntu.

1. **WEB-интерфейс**:
    1. Выбрать диск для увеличения через интерфейс
    1. Добавить объем:
        - LXC: **Volume actions → Resize**
        - VM: **Disk actions → Resize**

    Далее по тексту - для VM. Для LXC изменения применяются сразу.

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

## Using ZFS dataset as directory

ZFS dataset можно использовать как хранилище **Directory**:
1. В интерфейсе выбрать **Datacenter > Storage > Add > Directory**
1. Указать путь к *dataset*.

## Samba settings

- [Настройки в контейнере](#настройки-samba-в-контейнере)
- [Настройки на хосте](#настройки-samba-на-хосте)

---

### Настройки Samba в контейнере

Для работы с ACL необходимо предварительно установить пакет `acl`.

- [Создать группы пользователей](#create-groups-users-for-samba)
- [Настройка конфига Samba](#set-samba-settings)

    ---

1. <span id="create-groups-users-for-samba">Создать группы пользователей</span>:
    ```bash
    groupadd -g 2001 nas-ro
    groupadd -g 2002 nas-rw
    groupadd -g 2003 nas-admin
    ```

    - `nas-admin` - управление файлами (к примеру, через Cockpit).

        Если сделали это, то необходимо создать такого пользователя и добавить его в группу:
        ```bash
        adduser cockpitadmin
        usermod -aG nas-admin,sudo cockpitadmin
        ```

    - `nas-ro` - чтение через Samba.
    - `nas-rw` - чтение и запись через Samba.

    Пользователям Samba необходимо назначить необходимые права на чтение и запись. После [добавить для папок разрешения](#set-permissions-for-samba) для этих групп.

1. <span id="set-samba-settings">Настроить конфиг Samba `/etc/samba/smb.conf`</span>:

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

### Настройки Samba на хосте

- [Установка необходимых пакетов](#install-packages-for-samba)
- [Монтирование папок](#mount-dirs-for-samba)
- [Права доступа](#set-permissions-for-samba)

    ---

1. <span id="install-packages-for-samba">Установить пакет `acl`</span>:
    ```bash
    apt install acl -y
    ```

1. <span id="mount-dirs-for-samba">Подключить необходимые каталоги в контейнер</span>:
    ```bash
    pct set 120 -mp0 /mnt/storage,mp=/srv/storage
    pct set 120 -mp1 /mnt/readonly,mp=/mnt/readonly
    ```

    _где `120` - номер контейнера, `-mp0` - номер точки монтирования, `/mnt/storage` - каталог на хосте, `/srv/storage` - каталог в контейнере._
    _Подразумевается, что Readonly для группы пользователей, а не всего контейнера._

1. <span id="set-permissions-for-samba">Установить права доступа</span>:

    _На стандартном отображении UID непривилегированного LXC `UID 1000` внутри контейнера базово сдвигаетсся на 100000 и будет соответствовать `UID 101000` на хосте._

    - Только чтение:
        ```bash
        setfacl -m \
            u:101000:rwx,g:102003:rwx,g:102001:r-x \
            /mnt/readonly

        setfacl -m \
            d:u::rwx,d:g::r-x,d:g:102003:rwx,d:g:102001:r-x,d:m::rwx,d:o::--- \
            /mnt/readonly
        ```

    - Чтение и запись:
        ```bash
        setfacl -m \
            u:101000:rwx,g:102003:rwx,g:102001:r-x,g:102002:rwx \
            /mnt/storage

        setfacl -m \
            d:u::rwx,d:g::r-x,d:g:102003:rwx,d:g:102001:r-x,d:g:102002:rwx,d:m::rwx,d:o::--- \
            /mnt/storage
        ```
