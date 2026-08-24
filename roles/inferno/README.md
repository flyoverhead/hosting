# `inferno`

`inferno.name` hosting provider configuration

## Role variables

| Variable | Description | Example |
| :--- | :--- | :--- |
| `inferno_account` | inferno.name account configuration | Example in [defaults](defaults/main.yml) |
| `inferno_server` | VDS/VPS server configuration | Example in [defaults](defaults/main.yml) |

## Inventory

```yaml
inferno:
  hosts:
    ru:
```

## Playbook

```yaml
- name: inferno
  hosts: inferno
  gather_facts: true

  tasks:

    - name: inferno vps
      ansible.builtin.include_role:
        name: flyoverhead.hosting.inferno
        apply:
          become: false
          delegate_to: localhost
```

## Variables

```yaml

inferno_account:
  url: https://cp.inferno.name/api_client.php?action=
  cid: '' # account client id
  key: '' # account api key

inferno_server:
  ssh_key: id_ed25519
  template: debian12
  reinstall: false
```

## Facts set by this role

| Fact | Description |
| :--- | :--- |
| `inferno_template` | The os template id matching `inferno_server.template` |
| `inferno_order_id` | The order id of the server answering on `ansible_host` |
| `inferno_ssh_key_id` | The id of the account ssh key named `inferno_server.ssh_key` |
| `inferno_reinstall` | `inferno_server.reinstall` as the integer the API's `run` parameter expects |

## Notes

- This provider's API authenticates with a `cid` + `key` pair rather than the
  single bearer token `timeweb` and `vdsina` use, which is why
  `inferno_account` carries an extra field
- `reinstall: true` **destroys and rebuilds the server**, so the role is not
  idempotent in the ordinary sense — the reinstall is skipped entirely while
  `reinstall` is `false`, and performed unconditionally when it is `true`

## License

GPL-3.0

## Author Information

fLy0v3rH34d
