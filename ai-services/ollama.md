# Ollama install and configuration

## Ubuntu 26.04

1. Установить драйвера необходимые системные утилиты
    ```bash
    sudo apt install -y build-essential dkms linux-headers-generic
    ```
1. [Установить Ollama](https://ollama.com/download/linux) (драйвера Nvidia подтягиваются)
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    sudo reboot
    ```
1. **[Опционально]**: Модели можно хранить на отдельном диске
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
1. Скачать нужные модели командой `ollama pull имя_модели`
    - qwen3:4b - модель для теста
    - deepseek-r1:8b - рассуждение и логика
    - qwen2.5-coder:7b - для кода
    - llama3.1 - эрудиция
    - mistral-nemo - креативность
    - phi4 - интеллект для текстов
1. Разрешить подключение из сети к Ollama
    1. Открыть конфигурацию
        ```bash
        sudo systemctl edit ollama.service
        ```
    1. Добавить в пустом текстовом блоке (прочитайте комментарии в файле)
        ```ini
        [Service]
        Environment="OLLAMA_HOST=0.0.0.0"
        ```
    1. Применить настройки и перезапустить Ollama
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl restart ollama
        ```
1. Разрешить SSH, ограничить доступ к порту Ollama, включить Firewall
    ```bash
    sudo ufw allow ssh
    sudo ufw allow from 192.168.x.xx to any port 11434 proto tcp comment 'Разрешить Open WebUI' # Указать нужный ip-адрес (можно с маской) и комментарий
    sudo ufw deny 11434/tcp comment 'Блокировка внешнего доступа к Ollama API'
    sudo ufw reload
    ```

### Мониторинг видеокарты
- Сводка
    ```bash
    watch -n 1 nvidia-smi
    ```
- Построение графиков (необходима установка)
    ```bash
    nvtop
    ```

## Проверка после установки
После установки запустить скачанную модель и запустить мониторинг видеокарты:

```bash
ollama run имя_модели
```

Если при запросе к модели видеокарта нагружается, то все работает верно.

## Web UI

Для работы с языковыми моделями через веб-интерфейс можно [установить Open WebUI](open-web-ui.md#install-docker).

### Для Proxmox VE

1. Создать контейнер (CT) *Debian 13*
1. Обновить контейнер
     ```bash
    apt update && sudo apt full-upgrade -y
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
1. Установить *Open WebUI* через Docker
