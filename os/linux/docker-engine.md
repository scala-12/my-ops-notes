# Docker Engine

## Installation

Ниже не использованы команды `sudo`, т.к. инструкция для LXC контейнеров под root пользователем.
Если вы используете обычного пользователя, то добавьте пропущенные `sudo` по необходимости.

---

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
