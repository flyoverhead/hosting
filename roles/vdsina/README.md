# `flyoverhead.hosting.vdsina`

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

## Notes

- Servers are provisioned **key-only**. `config | create server` sends only
  `datacenter`, `name`, `server-plan`, `ssh-key` and `template`, so there is no
  root-password knob. `vdsina_server.password` was declared in an earlier
  version and never sent; it has been removed rather than left as a variable
  that silently does nothing, because a caller wiring a vaulted secret into it
  gained no effect and put the cleartext into anything rendering
  `vdsina_server`. If the create-server body ever gains a password field, add
  the variable back alongside it and set `no_log` on the tasks that touch it.

## Check mode

Every request in `detect.yml` is a read-only GET and carries `check_mode:
false`, so a check run performs them for real: `ansible.builtin.uri` declares no
check mode support at all, and a skipped result holds no `json` for the
`set_fact` after it to read. A check run therefore needs a working API key and
costs one call per read. It changes nothing on the provider side.

Nothing that changes the account runs, and a check run is correspondingly quiet
about it: creating the ssh key, ordering a server and dropping its backup
schedule are skipped as whole blocks, because each reads its result back out of
the creation response, and deleting a server (`state: absent`) is skipped as
well. What a check run does report is the `ansible_host` line it would write
into `host_vars`.

## Tags

| Tag | Purpose |
| :--- | :--- |
| `vdsina.detect` | Detect the datacenter, existing server, group/plan, ssh key and template ids `config` needs |
| `vdsina.config` | Create/update the server and ssh key, save `ansible_host`; also carried by `detect`'s task so a config-only run still has current facts |

This role is invoked with a *dynamic* `include_role` (required, because it
needs `delegate_to: localhost`). Ansible tag-filters a play's tasks when the
iterator initialises, so if the calling `include_role` task is not itself
tagged with the inner tag, `--tags vdsina.config` filters out the
`include_role` task itself and the role is never included at all — the inner
tags never get evaluated. The caller must carry the role's tags on the
`include_role` task itself, e.g.:

```yaml
- name: vdsina vps
  ansible.builtin.include_role:
    name: flyoverhead.hosting.vdsina
    apply:
      become: false
      delegate_to: localhost
  tags: [vdsina, vdsina.detect, vdsina.config]
```

or use the flat `vdsina` role tag instead.

## License

GPL-3.0

## Author Information

fLy0v3rH34d
