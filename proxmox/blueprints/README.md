# VMs and Containers

## Виртуальные машины
1. [*Windows 11*](vm-windows-11.md)
1. [*Ubuntu 26.04* VM для AI-сервисов](vm-ubuntu-26-ai.md)

## Контейнеры
1. [Контейнер для AI-сервиса](lxc-for-ai.md)
1. Шаринг файлов
    - [Простая Samba](lxc-samba.md)
    - [Cockpit + 45Drives plugin](lxc-cockpit-45drives.md)

## Замечания

### О LXC

1. **Cтатичный ip** обязателен, потому что роутер не может зафиксировать его сам.
1. **Debian 13** ругается, если не включить *nesting*.
1. Для безопасности лучше использовать *unprivileged* контейнеры.
