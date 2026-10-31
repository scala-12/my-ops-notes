# Dev Container в VS Code

Dev Container - это контейнер Docker, настроенный для разработки. VS Code открывает проект внутри контейнера: терминал, инструменты и расширения работают в его окружении, а файлы проекта остаются в рабочей папке.

## Что понадобится

- Docker-совместимый runtime. Ниже описаны Docker Desktop для Windows и Docker Engine для Ubuntu.
- Расширение **Dev Containers** от Microsoft (`ms-vscode-remote.remote-containers`) в VS Code.

## Установка среды

### Windows 11: Docker Desktop и WSL 2

1. Установите WSL 2 и дистрибутив Ubuntu. Если WSL ещё не установлен, откройте PowerShell от имени администратора и выполните:
   ```powershell
   wsl --install
   ```

   Перезагрузите компьютер, если этого потребует установщик. Проверьте доступные дистрибутивы и версию WSL командой `wsl --list --verbose`; у Ubuntu должна быть версия `2`.

1. Установите [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) и оставьте включённым backend на базе WSL 2.
1. В Docker Desktop откройте **Settings → Resources → WSL Integration** и включите интеграцию с тем дистрибутивом Ubuntu, в котором будете работать.
1. Установите VS Code и расширение **WSL** (`ms-vscode-remote.remote-wsl`), если ещё не установлено.
1. Откройте проект именно в окне VS Code, подключённом к WSL: выберите дистрибутив командой **WSL: Connect to WSL**, затем откройте или клонируйте проект внутри Linux-файловой системы, например в `~/projects/my-app`.

Для разработки через WSL рекомендуется хранить репозиторий в файловой системе Linux (`/home/...`), а не в `/mnt/c/...`: это обычно быстрее и надёжнее для файловых операций и наблюдения за изменениями. Команды Linux запускайте в окне VS Code, подключённом к WSL.

### Ubuntu

1. Установите Docker Engine и плагин Docker Compose по [официальной инструкции для Ubuntu](https://docs.docker.com/engine/install/ubuntu/).
1. Запустите Docker и проверьте его доступность:
   ```bash
   sudo systemctl enable --now docker
   sudo docker run --rm hello-world
   ```

1. Чтобы запускать Docker без `sudo`, добавьте своего пользователя в группу `docker`, затем заново войдите в систему:
   ```bash
   sudo usermod -aG docker "$USER"
   ```

   Членство в группе `docker` предоставляет привилегии, близкие к root. Если это неприемлемо, оставьте запуск Docker через `sudo` и настройте среду согласно политике вашей системы.

1. Установите VS Code для Linux и расширение **Dev Containers**. Откройте в VS Code папку проекта в Ubuntu.

## Добавление конфигурации в проект

1. Установите расширение **Dev Containers** через панель **Extensions** (`Ctrl+Shift+X`).
1. Откройте папку проекта в VS Code.
1. Откройте палитру команд (`Ctrl+Shift+P`) и выполните **Dev Containers: Add Dev Container Configuration Files...**.
1. Выберите подходящий шаблон для стека проекта. Если проект уже содержит `Dockerfile` или `compose.yaml`, выберите вариант конфигурации на их основе. VS Code создаст папку `.devcontainer` и файл `devcontainer.json`; в зависимости от шаблона также появятся `Dockerfile` или Compose-файл.
1. Проверьте созданную конфигурацию и сохраните её в Git вместе с проектом. Не добавляйте в неё пароли, токены и другие секреты.

Минимальный пример для проекта, которому достаточно базового Linux-окружения:

```json
{
  "name": "Development",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "remoteUser": "vscode"
}
```

Для реального проекта укажите нужную версию языка и системные зависимости: используйте подходящий шаблон, `Dockerfile` или [Features](https://containers.dev/features). Если проект использует базы данных или другие сервисы, настройте их через Docker Compose, а не устанавливайте вручную в контейнер после каждого запуска.

## Открытие проекта в контейнере

1. Убедитесь, что Docker запущен. В Windows запустите Docker Desktop и проверьте интеграцию с нужным WSL-дистрибутивом.
1. В VS Code откройте палитру команд и выполните **Dev Containers: Reopen in Container**.
1. При первом запуске VS Code соберёт или загрузит образ. Дождитесь окончания сборки; терминал и индикатор удалённого подключения покажут, что окно работает внутри контейнера.
1. Откройте терминал VS Code и проверьте окружение командами проекта, например `python --version`, `node --version` или `git status`.

Чтобы остановить работу, закройте окно или выполните **Dev Containers: Reopen Folder Locally**. Чтобы применить изменения в `devcontainer.json`, Dockerfile или Compose-конфигурации, используйте **Dev Containers: Rebuild and Reopen in Container**.

## Порты и расширения

- Добавьте нужные порты в `forwardPorts`, если приложение слушает порт внутри контейнера. VS Code обнаруживает многие порты автоматически, но явное указание делает конфигурацию понятнее.
- Расширения, которые должны работать в контейнере, можно перечислить в `customizations.vscode.extensions`. VS Code предложит установить их в удалённое окружение.
- Файлы проекта обычно доступны и с хоста, потому что рабочая папка подключается к контейнеру. Установленные внутри контейнера пакеты и системные настройки не заменяют файлы конфигурации проекта и могут исчезнуть при пересборке.

## Если контейнер не запускается

- Убедитесь, что Docker работает: `docker info` (в Ubuntu без настройки группы используйте `sudo docker info`).
- В Windows проверьте, что VS Code подключён к нужному дистрибутиву WSL, а Docker Desktop интегрирован именно с ним.
- Посмотрите **View → Output** и выберите канал **Dev Containers**: там обычно указана причина ошибки сборки или подключения.
- После изменения конфигурации пересоберите контейнер командой **Dev Containers: Rebuild and Reopen in Container**.
- При ошибках доступа к файлам в Ubuntu проверьте владельца файлов и права на папку проекта. Не исправляйте их рекурсивным назначением владельца наугад.

## Документация

- [Developing inside a Container](https://code.visualstudio.com/docs/devcontainers/containers) — подключение, запуск и работа в контейнере.
- [Create a Dev Container](https://code.visualstudio.com/docs/devcontainers/create-dev-container) — создание конфигурации.
- [Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/) и [Docker Engine на Ubuntu](https://docs.docker.com/engine/install/ubuntu/) — установка runtime.
