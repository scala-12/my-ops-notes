# Ollama install and configuration

## Ubuntu 26.04

### Установка Ollama

1. Установить драйвера необходимые системные утилиты:
    ```bash
    sudo apt install -y build-essential dkms linux-headers-generic
    ```

1. [Установить Ollama](https://ollama.com/download/linux) (драйвера Nvidia подтягиваются):
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    sudo reboot
    nvidia-smi # покажет состояние видеокарты
    ```

1. **[Опционально]**: Смена месторасположения моделей Ollama на заранее смонтированный диск
    1. Создайте папку для моделей в заранее смонтированный диск и откройте параметры Ollama
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

### Настройка firewall и доступа

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

1. Разрешить SSH, ограничить доступ к порту Ollama, включить Firewall:
    ```bash
    sudo ufw allow ssh
    sudo ufw allow from 192.168.x.xx to any port 11434 proto tcp comment 'Разрешить Open WebUI' # Указать нужный ip-адрес (можно с маской) и комментарий
    sudo ufw deny 11434/tcp comment 'Блокировка внешнего доступа к Ollama API'
    sudo ufw reload
    ```

## Мониторинг видеокарты
- Используйте `watch -n 1 nvidia-smi` для мониторинга состояния видеокарты.
- Установите `nvtop` для визуализации использования GPU.

## Проверка после установки
После установки запустить скачанную модель и запустить мониторинг видеокарты:

```bash
ollama run имя_модели
```

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

1. Создать контейнер (CT) *Debian 13*
1. Обновить контейнер
     ```bash
    apt update && apt full-upgrade -y
    reboot
    ```

1. Установить *Docker Engine*
    ```bash
    apt install -y ca-certificates curl
    install -m 0755 -d /etc/apt/keyrings

    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc

    tee /etc/apt/sources.list.d/docker.sources <<EOF
    Types: deb
    URIs: https://download.docker.com/linux/debian
    Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
    Components: stable
    Architectures: $(dpkg --print-architecture)
    Signed-By: /etc/apt/keyrings/docker.asc
    EOF

    apt update
    apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

1. Установить *Open WebUI* через Docker (Ссылка на инструкцию - выше)
