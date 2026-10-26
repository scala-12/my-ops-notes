# Open WebUI

## Install (Docker)

*[Актуальная инструкция](https://github.com/open-webui/open-webui#quick-start-with-docker) в репозитории*.

1. Установить и запустить **Open WebUI** в Docker; если Ollama на другом сервере, то указать его адрес (*OLLAMA_BASE_URL*)
    ```bash
    docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=http://192.168.x.x:11434 -v open-webui:/app/backend/data \
        --name open-webui \
        --restart always \
        ghcr.io/open-webui/open-webui:main
    ```
1. Открыть Open WebUI (*:3000*), создать учетную запись администратора

Если все установлено правильно, то в окне диалога будут показаны доступные модели.
- Если адрес Ollama указан неверно, его нужно исправить в настройках *Open WebUI*.
