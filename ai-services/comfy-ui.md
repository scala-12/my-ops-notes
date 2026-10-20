# ComfyUI installation

ComfyUI — графический интерфейс с узловыми workflow для генерации изображений. Open WebUI можно использовать как чат-интерфейс: пользователь описывает картинку обычным текстом, а Open WebUI отправляет запрос в ComfyUI.

## Ubuntu 26.04

Для системы потребуется более 16 ГБ внутренней памяти.

1. Установить драйвера Nvidia и необходимые пакеты (возможно нужны и libglx-mesa0 libglib2.0-0):
    ```bash
    sudo apt install -y ubuntu-drivers-common git
    sudo ubuntu-drivers install
    sudo reboot
    ```
1. Проверить статус видеокарты:
    ```bash
    nvidia-smi
    ```
1. Клонировать и перейти в проект *ComfyUI*
    ```bash
    git clone https://github.com/Comfy-Org/ComfyUI.git
    cd ComfyUI
    ```
1. Перенос хранилища моделей на другой раздел (папка должна быть создана с нужными правами пользователем до этого шага)
    ```bash
    mv ./models /mnt/storage/models
    ```
1. Создать виртуальное окружение 3.12
    - [**WARNING**] На момент написания рекомендовалось использовать Python 3.12. По умолчанию в Ubuntu 26 установлен 3.14. Поэтому придется установить виртуальное окружение нужной версии.
        1. Установить *uv*
            ```bash
            curl -LsSf https://astral.sh/uv/install.sh | sh
            source ~/.profile # обновить сессию терминала
            uv --version
            ```
        1. Создать в виртуальное окружение Python 3.12
            ```bash
            uv venv --python 3.12
            ```
        1. Далее вместо вызова **pip** использовать `uv pip`
1. Запустить локальное окружение
    ```bash
    source .venv/bin/activate
    ```
1. Установить зависимости для Nvidia
    ```python
    pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130
    ```
    - Проверьте, что PyTorch видит GPU (для CPU-установки результат `False` ожидаем):

        ```bash
        python -c "import torch; print('CUDA available:', torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
        ```
1. Установить зависимости проекта
    ```python
    pip install -r requirements.txt
    ```
1. Создать службу для автозапуска
    1. Создать файл **/etc/systemd/system/comfyui.service** с содержимым (замените ваш_юзер на имя пользователя в Ubuntu):
        ```ini
        [Unit]
        Description=ComfyUI AI Service
        After=network.target

        [Service]
        Type=simple
        User=ваш_юзер
        WorkingDirectory=/home/ваш_юзер/ComfyUI
        ExecStart=/home/ваш_юзер/ComfyUI/venv/bin/python3 /home/ваш_юзер/ComfyUI/main.py --listen 0.0.0.0 --port 8188
        Restart=always
        RestartSec=10

        [Install]
        WantedBy=multi-user.target
        ```
        - Чтобы модель не выгружалась в ОЗУ можно использовать ключ запуска `--highvram`
    1. Активируйте и запустите службу:
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl enable comfyui
        sudo systemctl start comfyui
        ```
1. Разрешить SSH, ограничить доступ к порту 8188, включить Firewall
    ```bash
    sudo ufw allow ssh # при изменении порта указать нужный
    sudo ufw allow from 192.168.xx.0/24 to any port 8188 proto tcp comment 'Доступ из локальной сети' # Указать подсеть для доступа
    sudo ufw deny 8188/tcp comment 'Блокировка внешнего доступа'
    sudo ufw reload
    ```
1. Открыть приложение на порту *:8188*

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
После установки запустить генерацию и запустить мониторинг видеокарты.

1. Выбрать модель из шаблонов
1. Скачать отсутствующие модели в указанные папки
    - У моделей подписаны ссылки и месторасположение.
    - По ссылке открыть хранилище, откуда можно скопировать ссылку на скачивание
    - Скачать каждую модель в нужную папку

Если при запросе к модели видеокарта нагружается, то все работает верно.

## Sources

- [ComfyUI: ручная установка](https://docs.comfy.org/installation/manual_install)
- [ComfyUI: параметры запуска сервера](https://docs.comfy.org/development/comfyui-server/startup-flags)
