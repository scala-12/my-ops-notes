# Ollama install and configuration

*В данной инструкции используется дистрибутив Ubuntu 26.04.*

1. [Установка Ollama](#установка-ollama)
1. [Настройка firewall и доступа](#настройка-firewall-и-доступа)
1. [Проверка после установки](#проверка-после-установки)
1. [Контекст и память](#контекст-и-память)
1. [Об Web UI](#web-ui)

---

## Установка Ollama

1. Установить драйвера необходимые системные утилиты:
    ```bash
    sudo apt install -y build-essential dkms linux-headers-generic
    ```

    При использовании LXC необходимо установить дополнительные пакеты:

    ```bash
    sudo apt install -y curl zstd
    ```

1. [Установить Ollama](https://ollama.com/download/linux) (драйвера Nvidia подтягиваются):
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    sudo reboot
    nvidia-smi # покажет состояние видеокарты
    ```

1. **[Опционально]**: Смена месторасположения моделей Ollama на заранее смонтированный диск
    1. Создайте папку для моделей, задайте права и откройте параметры Ollama:
        ```bash
        sudo mkdir -p /path/to/your/new/disk/ollama_models
        sudo chown -R ollama:ollama /path/to/your/new/disk/ollama_models
        sudo systemctl edit ollama.service
        ```

    1. Задайте новое расположение моделей (прочитайте комментарии в файле)
        ```ini
        [Service]
        Environment="OLLAMA_MODELS=/path/to/your/new/disk/ollama_models"
        ```

    1. Перезапустите службу
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl restart ollama.service
        ```

1. Скачать [нужные модели](text-models.md) командой `ollama pull имя_модели`

## Настройка firewall и доступа

1. Разрешить подключение из сети к Ollama:

    1. Открыть конфигурацию:
        ```bash
        sudo systemctl edit ollama.service
        ```

    1. Добавить в пустом текстовом блоке (внимание на комментарии в файле):
        ```ini
        [Service]
        Environment="OLLAMA_HOST=0.0.0.0"
        ```

    1. Применить настройки и перезапустить Ollama:
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl restart ollama
        ```

1. Ограничить доступ к порту Ollama, включить Firewall:

    **При использовании SSH**: Перед включением *firewall* удостовериться, что [настройки SSH](../os/linux/ssh-settings.md#настройка-firewall) корректны.
    ```bash
    sudo ufw allow from 192.168.x.xx to any port 11434 proto tcp comment 'Разрешить Open WebUI' # Указать нужный ip-адрес (можно с маской) и комментарий
    sudo ufw deny 11434/tcp comment 'Блокировка внешнего доступа к Ollama API'
    sudo ufw reload
    ```

## Проверка после установки
После установки запустить скачанную модель и запустить мониторинг видеокарты:

```bash
ollama run имя_модели
```

Для мониторинга видеокарты использовать следующие команды:
- Используйте `watch -n 1 nvidia-smi` для мониторинга состояния видеокарты.
- Установите `nvtop` для визуализации использования GPU.

Если при запросе к модели видеокарта нагружается, то все работает верно.

## Контекст и память

Размер контекста не определяется одним только названием модели. По умолчанию Ollama выбирает его с учётом доступной видеопамяти; для GPU с менее чем 24 ГБ VRAM документация Ollama указывает значение 4K. Больший контекст требует дополнительной памяти и может привести к частичному переносу вычислений на CPU или ошибке нехватки памяти.

Если нужно изменить контекст, задайте подходящее значение в настройках Ollama или в параметрах модели. Начните с умеренного значения и проверяйте фактическое использование командой:

```bash
ollama ps
```

В выводе можно посмотреть размер контекста и распределение модели между GPU и CPU. Увеличивайте контекст постепенно и только если системе хватает памяти.

## Web UI

Для работы с языковыми моделями через веб-интерфейс можно [установить Open WebUI](open-web-ui.md#install-docker).

### Для Proxmox VE

#### Контейнер с Open WebUI

1. Создать контейнер (CT) *Debian 13*
1. Обновить контейнер
     ```bash
    apt update && apt full-upgrade -y
    reboot
    ```

1. [Установить *Docker Engine*](../os/linux/docker-engine.md#installation)
1. Установить *Open WebUI* через Docker (Ссылка на инструкцию - выше)

#### Расшифровка аудио

1. [Создать контейнер](../proxmox/blueprints/lxc-for-ai.md) `whisper-gpu`:
    - **Disk size:** `16` GB
    - **Cores:** `4` (вся нагрузка пойдет на видеокарту, не обязательно выделять много).
    - **Memory:** `8` GB (нужно только для первичной распаковки модели в VRAM)
    - **Swap:** `1024` MiB
    - Подключить дополнительный диск `/mnt/storage` для моделей на 32 GB, без бекапа.
    - Пробросить видеокарту.

1. [Установить *Docker Engine*](../os/linux/docker-engine.md#installation).
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

    1. Установить пакеты NVIDIA Container Toolkit:
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

1. Настройка файрволла (укажите свою подсеть):
    ```bash
    ufw allow from 192.168.1.0/24 to any port 8000 proto tcp comment 'Доступ из локальной сети'
    ufw deny 8000/tcp comment 'Блокировка внешнего доступа'
    ufw reload
    ```

1. Запуск контейнера Speaches:
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
              - WHISPER__VAD_FILTER=true
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
        ```

    1. Запустить контейнер:
        ```bash
        docker compose up -d
        ```

    1. Скачать модель:
        ```bash
        curl -X POST http://localhost:8000/v1/models/<ИМЯ_МОДЕЛИ>
        ```

        Варианты моделей:
        - `Systran/faster-whisper-large-v3`
        - `deepdml/faster-whisper-large-v3-turbo-ct2`
        - `Systran/faster-whisper-medium`

1. Подключение к Open WebUI:
    1. Перейти в настройки **Open WebUI > Настройки > Инструменты > Аудио**.
    1. В разделе **Speech-to-Text** задать следующие параметры:
        - **STT Engine:** `OpenAI`
        - **API Base URL:** `http://<IP_нового_LXC>:8000/v1`
        - **API Key:** `sk-local`

            Контейнер speaches запускается с отключенной проверкой авторизации и принимает любой запрос.
            Надо ввести любой набор символов, потому что поле не должно быть пустым.

        - **STT Model:** `<название модели>`

        При переключении модели стоит перезагрузить Docker, чтобы выгрузить предыдущую модель:
        ```bash
        docker restart speaches
        ```

    1. Если необходимо указать язык речи по умолчанию, перейти в настройки **Open WebUI > Настройки > Предпочтения > Аудио** и укажите его.
