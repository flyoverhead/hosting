# `vdsina`

`VDSina` hosting provider configuration

## Role variables

| Variable | Description | Example |
| :--- | :--- | :--- |
| `vdsina_account` | VDSina account configuration | Example in [defaults](defaults/main.yml) |
| `vdsina_server` | VDS/VPS server configuration | Example in [defaults](defaults/main.yml) |

## Inventory

```yaml
vdsina:
  hosts:
    nl:
```

## Playbook

```yaml
- hosts: nl
  roles:
      - role: vdsina
```

## Variables

```yaml

vdsina_account:
  url: https://userapi.vdsina.com/v1
  api_key: <account_api_key>
  country: nl

vdsina_server:
  cpu: 1
  group: Standard servers
  name: nl-server
  password: <root_password> # The password must contain from 8 to 32 latin characters or numbers
  ssh_key: id_ed25519
  template: Debian 12
  state: present
```

## License

GPL-3.0

## Author Information

fLy0v3rH34d
