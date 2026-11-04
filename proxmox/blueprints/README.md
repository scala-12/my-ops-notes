# VMs and Containers

## Summary

1. [Установка *Windows 11* VM](vm-windows-11.md)
1. [Установка *Ubuntu 26.04* VM для AI-сервисов](vm-ubuntu-26-ai.md)
1. [Установка *Ubuntu 26.04* VM для AI-сервисов](lxc-ubuntu-26-ai.md)
1. Шаринг файлов
    - [Простая Samba](lxc-samba.md)
    - [Cockpit + 45Drives plugin](lxc-cockpit-45drives.md)

## Remarks

### LXC Remarks

1. **Cтатичный ip** обязателен, потому что роутер не может зафиксировать его сам.
1. **Debian 13** ругается, если не включить *nesting*.
1. Для безопасности лучше использовать *unprivileged* контейнеры.
