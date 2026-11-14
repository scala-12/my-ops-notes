# SSH settings

## Конфигурация SSH

1. В файле **/etc/ssh/sshd_config** настроить следующие параметры:
    - Сменить порт SSH: `Port новый_порт`.
    - Запретить вход под root: `PermitRootLogin no`

        При использовании LXC необходимо разрешить вход под root как для базового пользователя: `PermitRootLogin yes`

    - Разрешить вход по ключу: `PubkeyAuthentication yes`
    - Запретить вход по логину при использовании PubkeyAuth (_рекомендуется_): `PasswordAuthentication no`

1. При использовании `ssh.socket` и изменении порта SSH нужно добавить указанть новый порт `ListenStream=новый_порт` в конфигурации:
     ```bash
     systemctl edit ssh.socket
     ```

    После требуется перезагрузить сервис:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart ssh.socket
    ```

1. <span id="apply-ssh-config">Обновить конфигурацию SSH и перезагрузиться</span>:

    **Не обновлять конфигурацию, пока не** [добавлен public-ключ на сервер](#add-pubkey-to-server) и не [добавлен порт в Firewall](#ufw) (при его использовании).

    ```bash
    sudo systemctl restart ssh
    sudo reboot # почему-то порт не обновляется до перезагрузки
    ```

## Настройка Firewall

### UFW

Добавить правила Firewall с новым портом SSH (в примере `2222` и подсеть 192.168.1.0/24):
```bash
sudo ufw limit from 192.168.1.0/24 to any port 2222 proto tcp comment 'SSH из локальной'
sudo ufw deny 2222/tcp comment 'Блокировка внешнего SSH'
```
- `limit` - ограничить количество подключений с одного IP-адреса в минуту. Если лимит не нужен, то использовать `allow`.
- **Не применять правила, пока не** [добавлен public-ключ на сервер](#add-pubkey-to-server), иначе подключение будет невозможно.

После добавления ключа на сервер, можно применять правила:
```bash
sudo ufw enable
sudo systemctl is-enabled ufw   # проверка включен ли Firewall в автозагрузку
```

## Настройка SSH-клиентов

1. Сгенерировать ключ на клиенте, если его еще нет.

    **Windows** через *PuTTYgen*:
    - *Тип ключа* `EdDSA Ed25519`
    - Generate
    - Сохранить поле *Public key for pasting* в файл для сервера
    - Сохранить ключи как **id_ed25519** с соответствующими расширениями
    - Скопировать *Public key for pasting* для следующего шага

1. <span id="add-pubkey-to-server">Добавить public-ключ в файл **~/.ssh/authorized_keys** на сервере</span>

    Если файл не существовал, то добавить правильные права:

    ```bash
    chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys && chown -R $USER:$USER ~/.ssh
    ```

1. [Обновить конфигурацию SSH](#apply-ssh-config).

1. Подключение к серверу по SSH:

    **[Windows]**: Настроить использование ключей в PuTTY
    1. **Connection → SSH → Auth → Credentials**: выбрать ранее сгенерированный *Private key file*
    1. **Session**: Ввести адрес/порт и название, сохранить конфигурацию
    1. Подключиться
