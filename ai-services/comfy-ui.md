# ComfyUI installation

ComfyUI — это современный инструмент для создания графических интерфейсов с узловыми workflow для генерации изображений. Open WebUI предоставляет возможность использовать ComfyUI как чат-интерфейс для управления моделями. В этом руководстве описывается процесс установки ComfyUI на Ubuntu 26.04.

## Ubuntu 26.04

### Требования к системе
- **Объем оперативной памяти**: Для работы ComfyUI требуется более 16 ГБ RAM.

### Установка

1. Установить драйверы NVIDIA и Git (*возможно нужны и libglx-mesa0 libglib2.0-0*):
    ```bash
    sudo apt install -y ubuntu-drivers-common git
    sudo ubuntu-drivers install
    sudo reboot
    nvidia-smi # покажет состояние видеокарты
    ```

1. Клонировать репозиторий в домашнюю папку пользователя:
    ```bash
    cd ~
    git clone https://github.com/Comfy-Org/ComfyUI.git
    cd ComfyUI
    ```

1. [**Опционально**]: Перенос хранилища моделей на другой раздел; папка должна быть создана с нужными правами пользователем до этого шага:
    ```bash
    mv ./models /mnt/storage/models
    ```

1. Создать виртуальное окружение:
    - **Ubuntu 26**: На момент написания рекомендуется использовать Python 3.12. По умолчанию в Ubuntu 26 установлен 3.14. Поэтому придется установить виртуальное окружение нужной версии.
        1. Установить *uv*:
            ```bash
            curl -LsSf https://astral.sh/uv/install.sh | sh
            source ~/.profile # обновить сессию терминала
            uv --version
            ```
        1. Создать в виртуальное окружение Python 3.12:
            ```bash
            uv venv --python 3.12
            ```
        1. Далее вместо вызова **pip** использовать `uv pip`
    - **Ubuntu 24**: `python -m venv .venv`
1. Запустить виртуальное окружение:
    ```bash
    source .venv/bin/activate
    ```

1. Установить зависимости для Nvidia:
    ```bash
    pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130
    ```
    - Проверьте, что PyTorch видит GPU (для CPU-установки результат `False` ожидаем):

        ```bash
        python -c "import torch; print('CUDA available:', torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
        ```

1. Установить зависимости проекта:
    ```bash
    pip install -r requirements.txt
    ```

## Дополнительные настройки

### Создание службы systemd

1. Создать файл службы **/etc/systemd/system/comfyui.service** (замените ваш_юзер на имя пользователя):
    ```ini
    [Unit]
    Description=ComfyUI AI Service
    After=network.target

    [Service]
    Type=simple
    User=ваш_юзер
    WorkingDirectory=/home/ваш_юзер/ComfyUI
    ExecStart=/home/ваш_юзер/ComfyUI/venv/bin/python3 /home/ваш_юзер/ComfyUI/main.py --listen 0.0.0.0 --port 8188 --enable-manager
    Restart=always
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
    ```
    - Чтобы модель не выгружалась в ОЗУ можно использовать ключ запуска `--highvram`
    - Ключ `--enable-manager` для запуска *ComfyUI-Manager* для обновления нод.

1. Активировать и запустить службу:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable comfyui
    sudo systemctl start comfyui
    ```

### Настройка firewall и доступа

1. Настройте firewall:
    ```bash
    sudo ufw allow ssh # или указать нужный
    sudo ufw allow from 192.168.x.0/24 to any port 8188 proto tcp comment 'Доступ из локальной сети' # Указать подсеть
    sudo ufw deny 8188/tcp comment 'Блокировка внешнего доступа'
    sudo ufw reload
    ```

## Мониторинг видеокарты
- Используйте `watch -n 1 nvidia-smi` для мониторинга состояния видеокарты.
- Установите `nvtop` для визуализации использования GPU.

## Проверка после установки

1. Выберите модель:
    - Перейдите на вкладку "Templates" и выберите нужную модель.
2. Скачайте и установите модели:
    - Узнайте путь к модели и скачайте ее в соответствующую папку.

## Sources

- [ComfyUI: ручная установка](https://docs.comfy.org/installation/manual_install)
- [ComfyUI: параметры запуска сервера](https://docs.comfy.org/development/comfyui-server/startup-flags)
