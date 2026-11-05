# Integration into Home Assistant

1. **Datacenter → Permissions → Users**: Создать пользователя *haos-user* как *Proxmox user*
1. **Datacenter → Permissions → API Tokens**: Создать токен авторизации, записать серверный ключ - понадобится в *HA OS*
1. Добавить права (*Permissions*) для каждой сущности, которыми следует управлять, причем и для *пользователя*, и для *API Token*. К примеру, для управления состоянием VM/LXC требуется право **PVEVMUser**
1. Добавить в *HA OS* интеграцию **Proxmox VE**
    - Authentification: `Proxmox user`
    - Хост: `ip-адрес proxmox сервера`
    - Имя пользователя: `haos-user`
    - Порт: `8006` (если порт по умолчанию)
    - Use an API token: `✅`
    - Подтвердить
    - Token ID: `ИД токена API Token`
    - Secret: `Секрет при создании API Token`
    - Подтвердить

После создания должны быть показаны все сущности, к которым есть доступ. Для чтения состояний достаточно *PVEAuditor*.

К примеру, можно реализовать переключение VM, которые делят один ресурс.
