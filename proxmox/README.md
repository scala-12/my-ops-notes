# Proxmox VE guides

## Summary

1. [Начало работы с *Proxmox VE*](installation.md)
    - [Установка](installation.md#install)
    - [Сеть](installation.md#check-network)
    - [Подписки на репозитории](installation.md#repositories-settings-no-subscribtion-and-system-update)
    - [Хранилище и время](installation.md#time-and-storage)
    - [Пользовательский аккаунт](installation.md#user-account)
    - [Бекапы](installation.md#backup)
    - [Защита сетевого доступа](installation.md#restricting-network-access)
        - [Firewall](installation.md#firewall)
        - [Брутфорс](installation.md#brute-force-protection)

    - [Памятка](installation.md#памятка)
    - [Чек-лист после установки](installation.md#check-list)

1. [Использование мощностей видеокарты](gpu-use-cases.md)
    - [Проброс видеокарты в VM](gpu-use-cases.md#gpu-passthrough)
        - [Отмена проброса видеокарты](gpu-use-cases.md#revert-passthrough)

    - [Шаринг видеокарты с хоста](gpu-use-cases.md#host-gpu-sharing)
        - [Настройки на сервере](gpu-use-cases.md#host-settings)
        - [Настройки в контейнере](gpu-use-cases.md#container-settings)
        - [Откат шаринга с хоста](gpu-use-cases.md#revert-sharing)

1. [Инструкции по установке виртуальных машин и контейнеров](blueprints/README.md)
    - [Замечания](blueprints/README.md#remarks)

1. [Интеграция в Home Assistant](haos-integration.md)
1. [Сниппеты](snippets/README.md)
1. [Полезные утилиты](utils.md)
    - [Сброс пароля контейнера](utils.md#proxmox-ve-utils)

1. [Работа с дисками](disk-utils.md)
    - [Увеличение размера диска](disk-utils.md#resize-disk)
    - [Монтирование диска](disk-utils.md#mount-disk)
    - [Шаринг диска между VM](disk-utils.md#share-disk)
    - [Остановка шпинделя по таймауту](disk-utils.md#disk-stanby-by-timeout)
    - [Подключение ZFS датасета как Directory](disk-utils.md#using-zfs-dataset-as-directory)
