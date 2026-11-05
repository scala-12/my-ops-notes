# VMs and Containers

## Summary

1. [Установка *Windows 11* VM](vm-windows-11.md)
1. [Установка *Ubuntu 26.04* VM для AI-сервисов](vm-ubuntu-26-ai.md)
    - [Подготовка](vm-ubuntu-26-ai.md#preparing)
    - [Настройка](vm-ubuntu-26-ai.md#settings)
        - [Базовые](vm-ubuntu-26-ai.md#main)
        - [Настройка Public Key авторизации](vm-ubuntu-26-ai.md#pubkeyauth)

1. [Установка *Ubuntu 26.04* VM для AI-сервисов](lxc-ubuntu-26-ai.md)
    - [Подготовка](lxc-ubuntu-26-ai.md#preparing)
    - [Настройка](lxc-ubuntu-26-ai.md#settings)

1. Шаринг файлов
    - [Простая Samba](lxc-samba.md)
        - [Подготовка](lxc-samba.md#preparing)
        - [Настройка](lxc-samba.md#settings)

    - [Cockpit + 45Drives plugin](lxc-cockpit-45drives.md)
        - [Подготовка](lxc-cockpit-45drives.md#preparing)
        - [Настройка](lxc-cockpit-45drives.md#settings)

## Remarks

### LXC Remarks

1. **Cтатичный ip** обязателен, потому что роутер не может зафиксировать его сам.
1. **Debian 13** ругается, если не включить *nesting*.
1. Для безопасности лучше использовать *unprivileged* контейнеры.
