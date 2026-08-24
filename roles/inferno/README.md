# `flyoverhead.hosting.inferno`

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
| `inferno_ssh_key_exists` | Whether the account already holds a key named `inferno_server.ssh_key` |
| `inferno_ssh_key_id` | The id of that key — set from `detect` when it exists, from the upload when it did not |
| `inferno_reinstall` | `inferno_server.reinstall` as the integer the API's `run` parameter expects |

## Notes

- This provider's API authenticates with a `cid` + `key` pair rather than the
  single bearer token `timeweb` and `vdsina` use, which is why
  `inferno_account` carries an extra field
- `reinstall: true` **destroys and rebuilds the server**, so the role is not
  idempotent in the ordinary sense — the reinstall is skipped entirely while
  `reinstall` is `false`, and performed unconditionally when it is `true`
- `inferno_server.ssh_key` names the key in two places at once: the local
  public key is read from `~/.ssh/<name>.pub`, and the same string is the
  key's `name` in the inferno.name account. The role keeps the two in step —
  `config | add ssh keys` sends `name` explicitly on upload, and `detect.yml`
  selects on it — so a fresh account bootstraps without manual steps: `detect`
  finds nothing, sets `inferno_ssh_key_exists: false`, and `config` uploads the
  key and derives its id. Renaming the key in the provider's UI, or uploading
  it out of band under a different name, will break the lookup

## Tags

| Tag | Purpose |
| :--- | :--- |
| `inferno.detect` | Detect the os template, existing order, ssh key id and reinstall flag `config` needs |
| `inferno.config` | Upload the ssh key and reinstall the server; also carried by `detect`'s task so a config-only run still has current facts |

This role is invoked with a *dynamic* `include_role` (required, because it
needs `delegate_to: localhost`). Ansible tag-filters a play's tasks when the
iterator initialises, so if the calling `include_role` task is not itself
tagged with the inner tag, `--tags inferno.config` filters out the
`include_role` task itself and the role is never included at all — the inner
tags never get evaluated. The caller must carry the role's tags on the
`include_role` task itself, e.g.:

```yaml
- name: inferno vps
  ansible.builtin.include_role:
    name: flyoverhead.hosting.inferno
    apply:
      become: false
      delegate_to: localhost
  tags: [inferno, inferno.detect, inferno.config]
```

or use the flat `inferno` role tag instead.

## License

GPL-3.0

## Author Information

fLy0v3rH34d
