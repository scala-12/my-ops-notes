# LXC for AI

_Это не строгая инструкция, многие параметры можно изменить._

1. [Подготовка](#preparing)
1. [Настройка](#settings)

---

## Preparing

1. Подготовить видеокарту к [шарингу](../gpu-use-cases.md#host-gpu-sharing).
1. Создать LXC:
    - **General**:
        - Hostname: _Задать имя контейнера_
        - Uprivileged container: ✅ (для безопасности)
        - Nesting: ✅
        - Template: _Выбрать предварительно загруженный шаблон_

    - **Disk**:
        - Storage: `Выбрать`
        - Disk size: `32 ГБ`

    - **CPU > Cores**: _Подобрать в зависимости от задачи_
    - **Memory**:
        - Memory: _Подобрать в зависимости от задачи, но не всю_
        - Swap: `16384` MiB

    - **Network**: _статичный адрес_.

1. Если требуется работа с видеокартой, добавить [настройки контейнера](../gpu-use-cases.md#container-settings).
1. Запустить контейнер.

## Settings

1. При надобности [подключить дополнительный диск](../disk-utils.md#lxc-mount-point).
1. Обновить контейнер:
    ```bash
    sudo apt update && sudo apt upgrade -y
    nvidia-smi  # если добавили работу с видеокартой - проверить работоспособность
    ```

1. Если скорость загрузки низкая, то отключить оптимизации для сетевого интерфейса:
    ```bash
    apt update && apt install -y ethtool
    ethtool -K eth0 tx off rx off gso off tso off gro off
    ```

    TODO: проверить детальнее

1. [Настроить SSH](../../os/linux/ssh-settings.md)
