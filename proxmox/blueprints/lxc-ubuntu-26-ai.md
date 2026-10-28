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

1. Настроить SSH:

    В файле **/etc/ssh/sshd_config**
    - Сменить порт SSH: `Port новый_порт`.
    - Разрешить вход под root: `PermitRootLogin yes` (у контейнера базовый пользователь root)
    - Разрешить вход по ключу: `PubkeyAuthentication yes`
    - Запретить вход по логину: `PasswordAuthentication no`

1. Добавить правила Firewall с новым портом SSH (в примере `2222` и подсеть 192.168.1.0/24):
    ```bash
    sudo ufw allow from 192.168.1.0/24 to any port 2222 proto tcp comment 'SSH из локальной'
    sudo ufw deny 2222/tcp comment 'Блокировка внешнего SSH'
    ```

1. Сгенерировать ключ на клиенте через *PuTTYgen*, если его еще нет.

    **Windows**

    - *Тип ключа* `EdDSA Ed25519`
    - Generate
    - Сохранить поле *Public key for pasting* в файл для сервера
    - Сохранить ключи как **id_ed25519** с соответствующими расширениями

1. Скопировать *Public key for pasting* в **~/.ssh/authorized_keys** на сервере
1. Задать правильные права
    ```bash
    chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys && chown -R $USER:$USER ~/.ssh
    ```

1. Обновить конфигурацию SSH и перезагрузиться:
    ```bash
    sudo systemctl restart ssh
    sudo reboot # почему-то порт не обновляется до перезагрузки
    ```

1. **[Windows]**: Настроить использование ключей в PuTTY
    1. **Connection → SSH → Auth → Credentials**: выбрать ранее сгенерированный *Private key file*
    1. **Session**: Ввести адрес/порт и название, сохранить конфигурацию
    1. Подключиться
