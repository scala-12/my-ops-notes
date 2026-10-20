# Ollama

## Ollama core install

1. Установить драйвера необходимые системные утилиты
    ```bash
    sudo apt install -y build-essential dkms linux-headers-generic
    ```
1. Установить **Ollama** (Nvidia подтягиваются)
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    sudo reboot
    ollama pull qwen3:4b # модель для теста
    ollama pull deepseek-r1:8b # рассуждение и логика
    ollama pull qwen2.5-coder:7b # для кода
    ollama pull llama3.1 # эрудиция
    ollama pull mistral-nemo # креативность
    ollama pull phi4 # интеллект для текстов
    ollama pull llama3-groq-tool-use # для phi4
    ```
1. Разрешить подключение из сети к *Ollama*
    ```bash
    sudo systemctl edit ollama.service
    ```
    Добавить в пустом текстовом блоке:
    ```ini
    [Service]
    Environment="OLLAMA_HOST=0.0.0.0"
    ```
    Применить настройки и перезапустить Ollama:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart ollama
    ```
1. Включить **Firewall**, разрешить SSH, ограничить доступ к порту *Ollama*
    ```bash
    sudo ufw allow ssh
    sudo ufw allow from 192.168.x.xx to any port 11434 proto tcp comment 'Разрешить Open WebUI'
    sudo ufw deny 11434/tcp
    sudo ufw reload
    ```

1. Для мониторинга видеокарты используется команда:
    ```bash
    watch -n 1 nvidia-smi
    ```
    Для построения графиков:
    ```bash
    sudo apt install -y nvtop
    nvtop
    ```

## Web UI

Для работы с языковыми моделями будет использоваться **Open WebUI**

1. **[Для Proxmox]** Создать контейнер (CT) *Debian 13*

1. [Установить Open WebUI](open-web-ui.md#install)
