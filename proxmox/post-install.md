# Proxmox VE Post install

## Настройка репозиториев обновления

1. Отключить **enterprices** репозитории из обновлений
1. Добавить вместо отключенных репозиториев **No-subscribtion**-версии
    - No-subscribtion
    - Cept Squid No-subscribtion
1. Отключить проверку подписки (уведомление при входе)
    1. Открыть **/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js** 
    1. Найти блок кода с проверкой подписки или строчку с `No valid subscription`
    1. Закомментировать строку с показом модалки `No valid subscription`
