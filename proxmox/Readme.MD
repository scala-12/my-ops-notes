# Proxmox VE

## Настройка

1. Отключить **enterprices** репозитории из обновлений
1. Добавить вместо отключенных репозиториев **No-subscribtion**-версии
    - No-subscribtion
    - Cept Squid No-subscribtion
1. Отключить проверку подписки (уведомление при входе)
    1. Открыть **/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js** 
    1. Найти блок кода с проверкой подписки или строчку с `No valid subscription`
    1. Закомментировать строку с показом модалки `No valid subscription`
    1. Перезапустить веб-интрефейс коммандой `systemctl restart pveproxy.service`

## Установка Windows 11

1. Добавить ISO Windows 11 PRO (с RDP) и [драйверов](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso)
1. Создать VM
    - **OS**: выбрать ISO, тип ОС
    - **System**: выбрать EFI диск
    - **Disk**: Disk size *64GB*, Cache *Write back*, *Discard*, *IO thread*
    - **CPU**: Cores - реальное число ядер, type *host*
    - **Memory**: более 16ГБ
    - **Network**: Model VirtIO (paravirtualized)
1. Запустить VM, открыть консоль, установить Windows
    - На этапе выбора диска нажать "Загрузить" для установки драйверов.
    - Для установки без сети: консоль `Shift + F10`, команда `oobe\bypassnro`
    - Установить дополнения гостевой ОС
