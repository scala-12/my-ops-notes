# Open WebUI

## Install

Установка через Docker

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
