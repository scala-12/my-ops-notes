# Ollama

## Ubuntu 24.04

1. Установить драйвера необходимые системные утилиты
    ```bash
    sudo apt install -y build-essential dkms linux-headers-generic
    ```
1. [Установить Ollama](https://ollama.com/download/linux) (драйвера Nvidia подтягиваются)
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    sudo reboot
    ```
1. Скачать нужные модели командой `ollama pull модель`
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
    1. Добавить в пустом текстовом блоке
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
    sudo ufw allow from 192.168.x.xx to any port 11434 proto tcp comment 'Разрешить Open WebUI'
    sudo ufw deny 11434/tcp
    sudo ufw reload
    ```

### Для мониторинга видеокарты
- Сводка
    ```bash
    watch -n 1 nvidia-smi
    ```
- Построение графиков: `nvtop` (необходимо установить).

## Web UI

Для работы с языковыми моделями будет использоваться **Open WebUI**

1. **[Для Proxmox]** Создать контейнер (CT) *Debian 13*

1. [Установить Open WebUI](open-web-ui.md#install)
