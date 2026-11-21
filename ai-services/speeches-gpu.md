# Speeches GPU

_Локальный самостоятельный локальный сервер для расшифровки аудио, который работает по стандартному протоколу OpenAI Audio API._

## Установка

В примерах ниже используется **Debian 13** с видеокартой Nvidia под пользователем `root`.

1. **[Proxmox VE]:** [Создать контейнер](../proxmox/blueprints/lxc-for-ai.md) `whisper-gpu`:
    - **Disk size:** `16` GB
    - **Cores:** `4` (вся нагрузка пойдет на видеокарту, не обязательно выделять много).
    - **Memory:** `8` GB (нужно только для первичной распаковки модели в VRAM)
    - **Swap:** `1024` MiB
    - Подключить дополнительный диск `/mnt/storage` для моделей на 32 GB, без бекапа.
    - Пробросить видеокарту.

1. [Установить _Docker Engine_](../os/linux/docker-engine.md#installation).
1. Установить [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
    1. Установить необходимые пакеты:
        ```bash
        apt update && apt install -y --no-install-recommends \
            ca-certificates \
            curl \
            gnupg2
        ```

    1. Подключите репозиторий:
        ```bash
        curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
            && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
        ```

    1. Установить пакеты _NVIDIA Container Toolkit_:
        ```bash
        apt update
        export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.20.2-1
        apt-get install -y \
            nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
            nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
            libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
            libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
        ```

    1. Отключение cgroups внутри непривилегированного LXC для предотвращения ошибки доступа к GPU:
        ```bash
        nvidia-ctk runtime configure --runtime=docker
        sed -i 's/^#no-cgroups = false/no-cgroups = true/' /etc/nvidia-container-runtime/config.toml
        systemctl restart docker
        ```

    Для проверки работоспособности GPU в Docker можно использовать следующую команду:
    ```bash
    docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
    ```
    Если все правильно, будет выведена таблица с информацией о GPU.

1. Настройка файрволла (_укажите свою подсеть_):
    ```bash
    ufw allow from 192.168.1.0/24 to any port 8000 proto tcp comment 'Доступ из локальной сети'
    ufw deny 8000/tcp comment 'Блокировка внешнего доступа'
    ufw reload
    ```

1. Запуск контейнера _Speaches_:
    1. Создайте файл `/opt/speaches/docker-compose.yml`.

        В строке с указанием портов замените `192.168.1.50` на IP-адрес вашего сервера.
        Это позволит корректно настраивать фаерволл и настроить доступ к контейнеру.

        ```bash
        mkdir -p /opt/speaches && cd /opt/speaches
        cat << 'EOF' > docker-compose.yml
        services:
          speaches:
            image: ghcr.io/speaches-ai/speaches:latest-cuda
            container_name: speaches
            restart: unless-stopped
            ports:
              - "192.168.1.50:8000:8000"
            volumes:
              - /mnt/storage/hf-cache/hub:/home/ubuntu/.cache/huggingface/hub
            environment:
              - HF_HOME=/home/ubuntu/.cache/huggingface
              - WHISPER__INFERENCE_DEVICE=cuda
              - WHISPER__COMPUTE_TYPE=float16
              - WHISPER__VAD_FILTER=true                    # Отсекает тишину, музыку и фоновые шумы до передачи аудио в нейросеть
              - WHISPER__CONDITION_ON_PREVIOUS_TEXT=false   # Отключает предсказание текста на основе предыдущего
              - WHISPER__REPETITION_PENALTY=1.1             # Уменьшает вероятность повторения слов

            deploy:
              resources:
                reservations:
                  devices:
                    - driver: nvidia
                      count: 1
                      capabilities: [gpu]
        EOF
        ```

        При ошибке можно выполнить `docker compose down && docker compose up -d` для сброса контейнера и повторной сборки.

    1. Создать папку моделей, настроить права на запись:
        ```bash
        mkdir /mnt/storage/hf-cache/hub -p
        chmod -R 777 /mnt/storage/hf-cache
        chown -R 1000:1000 /mnt/storage/hf-cache
        ```

    1. Запустить контейнер:
        ```bash
        docker compose up -d
        ```

    1. Скачать модель:
        ```bash
        curl -X POST http://localhost:8000/v1/models/<ИМЯ_МОДЕЛИ>
        ```

        | Модель | Параметры | Скорость | Особенности |
        | --- | --- | --- | --- |
        | `Systran/faster-whisper-large-v3` | 1,55 млрд | Низкая | Лучше справляется со сложным аудио, акцентами и редкими языками. Склонна к «галлюцинациям» на длинных паузах (нужен VAD). |
        | `deepdml/faster-whisper-large-v3-turbo-ct2` | 809 млн | Высокая (в 4–6 раз быстрее v3) | Урезанный декодер при сохранении энкодера от large-v3. Почти не уступает ей в качестве на русском и других языках, но работает радикально быстрее. Примечание: не умеет переводить речь на английский. |
        | `guillaumekln/faster-whisper-large-v2` | 1,55 млрд | Низкая (как v3) | Стабильный предшественник. Иногда дает меньше галлюцинаций на тишине и музыке, чем large-v3, но хуже работает с редкими языками и сложным шумом. |
        | `Systran/faster-distil-whisper-large-v3` | 756 млн | Очень высокая (в 5–6 раз быстрее v3) | Строго для английского языка. На русском и других неанглийских языках работает плохо и выдает ошибки. |
        | `Systran/faster-whisper-medium` | 769 млн | Средняя | Устаревающий середнячок. По весу и памяти сопоставима с large-v3-turbo, но заметно проигрывает ей в точности распознавания (особенно для русского языка). |
        | `Systran/faster-whisper-small` | 244 млн | Максимальная | Для слабых ПК и CPU. Подходит для чистой речи без сложной терминологии, когда важнее скорость на процессоре без видеокарты, чем идеальная точность. |

    При переключении модели стоит перезагрузить Docker, чтобы выгрузить предыдущую модель:
    ```bash
    docker restart speaches
    ```

При успешной установке с сервером можно будет работать по `<ip адрес>:8000`.
