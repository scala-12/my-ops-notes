# Ubuntu 26.04 for AI

## Preparing

1. Подготовить видеокарту к [шарингу](../gpu-use-cases.md#host-gpu-sharing).
1. Добавить Container template.
1. Создать LXC:
    - **General**:
        - Hostname: *Задать имя контейнера*
        - Uprivileged container: ✅ (для безопасности)
        - Nesting: ✅
        - Template: *Выбрать шаблон*

    - **Disk**:
        - Storage: `Выбрать`
        - Disk size: `32 ГБ`

    - **CPU > Cores**: *Половину ядер*
    - **Memory**:
        - Memory: *больше - лучше, но не всю*
        - Swap: `16384` MiB

    - **Network**: сделать статичный адрес.

1. Добавить [настройки контейнера](../gpu-use-cases.md#container-settings) для работы с видеокартой.
1. Запустить контейнер.

## Settings

1. На хосте выбрать контейнер, перейти в *Resources* и добавить точку монтирования. Нужно выбрать *Storage*, размер и указать путь монтирования в контейнере.

    Далее по тексту - внутри контейнера.

1. Обновить контейнер:
    ```bash
    sudo apt update && sudo apt upgrade -y
    nvidia-smi
    ```

1. Если скорость загрузки низкая, то отключить оптимизации для сетевого интерфейса:
    ```bash
    apt update && apt install -y ethtool
    ethtool -K eth0 tx off rx off gso off tso off gro off
    ```

1. [Настроить SSH](../../os/linux/ssh-settings.md)
