# Proxmox VE GPU sharing

## GPU Passthrough

1. Добавить в загрузчик **/etc/default/grub** параметры для ЦП:
    - **Intel**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"`
    - **AMD**: `GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"`

    Параметр `iommu=pt` (Pass-Through) переводит систему в режим, когда процессор не трогает устройства и не тратит ресурсы на их виртуализацию, пока его об этом явно не попросят.

    Параметр `intel_iommu=on` / `amd_iommu=on` активирует на уровне процессора технологию IOMMU (Intel VT-d / AMD-Vi).

1. Обновить загрузчик
    ```bash
    update-grub
    ```

1. Добавить модули vfio в файле **/etc/modules-load.d/vfio.conf**:
    ```apacheconf
    vfio
    vfio_iommu_type1
    vfio_pci
    ```

    *VFIO* - это специальный "виртуальный сейф". Когда устройство попадает в этот сейф, обычная операционная система теряет к нему доступ, а гипервизор KVM получает возможность передать это устройство внутрь виртуальной машины «как есть».

1. В файле **/etc/modprobe.d/pve-blacklist.conf** запретить на хосте загрузку драйверов GPU (Nvidia и AMD):
    ```apacheconf
    blacklist nouveau
    blacklist nvidia*
    blacklist amdgpu
    blacklist radeon
    ```

1. Импортировать модули
    ```bash
    update-initramfs -u -k all
    ```

1. Отключить инициализацию видеокарты на хосте
    1. Найти ID GPU `lspci` вида `xx:xx`
    1. Скопировать из результата команды `lspci -n -s xx:xx` (ID GPU с предыдущего шага) ID вида `xxxx:xxxx`
    1. Добавить в **/etc/modprobe.d/vfio.conf** строку `options vfio-pci ids=xxxx:xxxx,yyyy:yyyy disable_vga=1`, подставив данные с предыдущего шага

1. Перезагрузить, проверить работу IOMMU
    - **Intel**: вывод `dmesg | grep -e DMAR -e IOMMU` содержит *IOMMU enabled*
    - **AMD**: вывод `dmesg | grep -e DMAR -e IOMMU -e AMD-Vi` содержит *interrupt remapping*

1. В VM добавить *PCI Device*, чаще всего как
    - All Functions: `✅`
    - ROM-Bar: `✅`
    - PCI-Express: `✅`

    *Primary GPU* используется для вывода изображения через это устройство (скорее всего, нужно отключить видеокарту в настройках VM).

### Revert passthrough

Для отмены блокировки видеокарты нужно откатить часть внесенных изменений:

1. В файле **/etc/modprobe.d/pve-blacklist.conf** закоментировать добавленные строки, кроме `blacklist nouveau`, и выполнить команду:
    ```bash
    update-initramfs -u -k all
    ```
1. В файле **/etc/modprobe.d/vfio.conf** закомментировать добавленную строку.
1. Перезагрузить систему.

## Host GPU sharing

### Host settings

1. Отменить [GPU-passthrough](#revert-passthrough)
1. Установить необходимые программы:
    ```bash
    apt install build-essential pve-headers-$(uname -r) python3
    ```

1. Скачать драйвера с сайта Nvidia, добавить файлу разрешение на выполнение `chmod +x` и выполнить файл:
    - Выбрать проприетарный драйвер.
    - Подписать драйвер.
    - Сгенерировать новую пару ключей.
    - Удалить приватный ключ.
    - Сохранить адрес ключа.
    - Установить подписанный модуль.
    - Согласиться с предупреждениями.
    - Дождаться установки.
    - Отказаться от запуска утилиты `nvidia-xconfig`.

1. Зарегистрировать ключ в Secure boot:
    1. Импортировать в базу MOK сгенерированный ключ:
        ```bash
        mokutil --import /usr/share/nvidia/nvidia-modsign-crt-XXXXXXXX.der
        ```

    1. Ввести временный пароль (запомните его).
    1. Подключить к серверу монитор и клавиатуру, перезагрузиться.
    1. В загрузившемся **MOK Manager** выбрать `Enroll MOK` > `Continue` > *Enroll the key(s)?* `Yes` > Ввести временный пароль > `Reboot`.

1. Создать сервис для автоматической инициализации видеокарты:

    Создать файл сервиса `/etc/systemd/system/nvidia-init.service` с содержимым:
    ```ini
    [Unit]
    Description=Initialize NVIDIA Devices
    After=network.target

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/nvidia-smi
    RemainAfterExit=yes

    [Install]
    WantedBy=multi-user.target
    ```

    Активировать сервис:
    ```bash
    systemctl daemon-reload
    systemctl enable nvidia-init.service
    systemctl start nvidia-init.service
    ```

1. Узнать реальные GID групп на хосте (запомнить их):
    ```bash
    grep -E '^(video|render):' /etc/group
    ```

1. Разрешаем маппинг этих групп непривиленированным контейнерам, добавив в конец строки с GID:

    `/etc/subgid`
    ```
    root:44:1
    root:993:1
    ```

    , где `44` и `993` - GID групп *video* и *render*.

### Container settings

1. Узнать реальные GID групп на хосте (запомнить их):
    ```bash
    grep -E '^(video|render):' /etc/group
    ```

1. На хосте определить *номера устройств* (запомнить их):

    - **Nvidia**: в строках */dev/nvidia0* и */dev/nvidia-uvm* первое число после *crw-rw-rw- 1 root root*:
        ```bash
        ls -l /dev/nvidia*
        ```

        Могут быть числа `195` и `510`. Далее в примерах используются они.

1. Добавить строки в конфигурационный файл контейнера со значениями, полученными на предыдущих шагах:

    Далее GID Video `44`, render `993`; при других значениях надо пересчитывать смещения.

    Также внутрь прокинут драйвер Nvidia c хоста - может не соответствовать версия утилит.

    `/etc/pve/lxc/101.conf` (*101* - номер контейнера)
    ```apacheconf
    # 1. Стандартный проброс файлов NVIDIA с номерами устройств
    lxc.cgroup2.devices.allow: c 195:* rwm
    lxc.cgroup2.devices.allow: c 510:* rwm
    lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
    lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
    lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
    lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file

    # 2. Правила маппинга ID (UID/GID)
    # Маппинг пользователей (UID) остается стандартным
    lxc.idmap: u 0 100000 65536
    # Маппинг групп (GID): разбитый на диапазоны под video (44) и render (933)
    lxc.idmap: g 0 100000 44
    lxc.idmap: g 44 44 1
    lxc.idmap: g 45 100045 948
    lxc.idmap: g 993 993 1
    lxc.idmap: g 994 100994 64542
    ```

    **Как работает**:
    | Строка | Объяснение |
    | --- | --- |
    | `g 0 100000 44` | Берем первые 44 группы (от 0 до 43) и смещаем их на 100000 |
    | `g 44 44 1` | Ровно 44-ю группу пробрасываем один в один |
    | `g 45 100045 948` | Берем группы от 45 до 992 (всего 948 штук) и смещаем на 100000 |
    | `g 993 993 1` | Ровно 993-ю группу пробрасываем один в один |
    | `g 994 100994 64542` | Забираем все оставшиеся группы от 994 до 65535 (всего 64542 штуки) и смещаем на 100000 |

1. Запустить контейнер и установить тот же драйвер, что и на хосте, но с ключом `--no-kernel-module`

    Этот флаг критически важен, так как контейнер использует ядро хоста и не может компилировать модули ядра самостоятельно.

    Для копирования с хоста файла установки на запущенный контейнер можно использовать команду, подставив свои значения:

    ```bash
    pct push 101 /path/to/original /path/to/copy
    ```

    В контейнере для запуска требуется выставить права `chmod +x`.

**Никогда не обновляйте драйвер NVIDIA внутри контейнера отдельно от хоста.**

### Revert sharing

1. Для отмены изменений необходимо отключить службу инициализации видеокарты:
    ```bash
    systemctl disable --now nvidia-init.service
    systemctl daemon-reload
    ```

1. Удалить драйвер NVIDIA с хоста Proxmox:
    ```bash
    nvidia-uninstall
    ```

1. Внесите драйверы NVIDIA в черный **/etc/modprobe.d/blacklist-nvidia.conf** и обновить initramfs `update-initramfs -u -k all` как в passthrough-методе.
1. Отключить автоматический запуск контейнеров и не запускать их, или удалить добавленные в его конфигурацию строки `lxc.cgroup2.devices.allow` и `lxc.mount.entry`
1. Перезагрузиться.
