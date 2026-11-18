# Proxmox VE utils

1. [Сброс пароля LXC](#сброс-пароля-lxc)

---

## Сброс пароля LXC

```bash
pct exec 100 -- passwd root
```

Где 100 - id контейнера.
