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
- name: vdsina
  hosts: vdsina
  gather_facts: true

  tasks:

    - name: vdsina vps
      ansible.builtin.include_role:
        name: flyoverhead.hosting.vdsina
        apply:
          become: false
          delegate_to: localhost
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

## Facts set by this role

| Fact | Description |
| :--- | :--- |
| `vdsina_datacenter_id` | The datacenter id matching `vdsina_account.country` |
| `vdsina_server_exists` | Whether a server already answers on `ansible_host` |
| `vdsina_server_id` | The id of that existing server, when `vdsina_server_exists` |
| `vdsina_server_group_id` | The id of the server group matching `vdsina_server.group` |
| `vdsina_server_plan_id` | The id of the server plan matching `vdsina_server_group_id` and `vdsina_server.cpu` |
| `vdsina_ssh_key_exists` | Whether `vdsina_server.ssh_key` is already registered with the account |
| `vdsina_ssh_key_id` | The id of that ssh key, whether pre-existing or just created |
| `vdsina_template_id` | The template id matching `vdsina_server.template` |

## License

GPL-3.0

## Author Information

fLy0v3rH34d
