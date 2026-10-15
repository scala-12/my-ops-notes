# AI-infrastructure

## AI-Server
Для работы с языковыми моделями будет использоваться **Ollama**.

### Настроки VM

- **System**: 
    - Machine *q35*,
    - BIOS *OVMF (UEFI)*,
    - Pre-Enroll Keys ❌,
    - Qemu Agent ✅.
- **Disk**:
    - Выбрать Storage,
    - Disk size *96GB*,
    - Cache *Write back*,
    - Discard ✅,
    - IO thread ✅;
    - SSD emulation ✅,
    - Backup ❌.
- **CPU**:
    - Cores *реальное число ядер*,
    - Type *host*.
- **Memory**:
    - Memory *больше - лучше*,
    - Balooning Device ❌.
- **Network**: Model *VirtIO (paravirtualized)*
- **Перед запуском** пробросить видеокарту как PCI Device:
    - All Functions ✅,
    - ROM-Bar ✅,
    - PCI-Express ✅.

### Установка дистрибутива

 1. Базовый вариант установки, не *minimized*.
 1. *Set up this disk as an LVM goup* ❌. 
 1. *Install OpenSSH server* ✅.

**После установки** удалить ISO-образ Ubuntu из списка устройств. Дальше удобнее подключаться по SSH.

 1. Обновить систему
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo reboot
    ```
1. Установить драйвера необходимые системные утилиты
    ```bash
    sudo apt install -y build-essential dkms linux-headers-generic
    ```
1. Установить **Ollama** (Nvidia подтягиваются)
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    sudo reboot
    ollama run qwen3:4b # модель для теста
    ollama run deepseek-r1:8b # рассуждение и логика
    ollama run qwen2.5-coder:7b # для кода
    ollama run llama3.1 # эрудиция
    ollama run mistral-nemo # креативность
    ollama run phi4 # интеллект для текстов
    ollama run llama3-groq-tool-use # для phi4
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

1. Создать контейнер (CT) Debian
1. Обновить, установить **Docker Engine**
    ```bash
    apt update && sudo apt full-upgrade -y
    reboot
    
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
1. Установить **Open WebUI** в Docker
    ```bash
    docker run -d -p 3000:8080 \
        -e OLLAMA_BASE_URL=http://192.168.x.x:11434 \
        -v open-webui:/app/backend/data \
        --name open-webui \
        --restart always \
        ghcr.io/open-webui/open-webui:main
    ```
1. Открыть Open WebUI (*:3000*), создать учетную запись администратора

## Image generator
