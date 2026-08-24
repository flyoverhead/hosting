# Changelog

All notable changes to `flyoverhead.hosting`.

## Unreleased

### Fixed

- **Check mode**: `ansible-playbook --check --diff` now completes for all three
  roles. Every `detect.yml` read provider state with `ansible.builtin.uri`,
  which declares no check mode support and is therefore skipped in a check run,
  and the `set_fact` immediately after it dereferenced `.json` -- so the play
  aborted on the first task of every role. All fourteen read-only GETs now carry
  `check_mode: false`.

### Changed

- **inferno**, **timeweb**, **vdsina**: the blocks that change the account --
  uploading an ssh key, ordering a server, dropping its backup schedule,
  deleting a server -- are now skipped as whole blocks in a check run, since
  each reads its own result back out of the creation response. `inferno`
  additionally sets `inferno_ssh_key_id` to `check-mode-placeholder`, because
  `config | reinstall vps` has its `url` templated before the `uri` action gets
  to skip itself.
- A check run now makes real read-only API calls, so it needs working
  credentials and costs one call per read. It changes nothing on the provider
  side.
- Every role README gained a `## Check mode` section stating what a check run
  covers and what it cannot.

## 1.0.0

Initial release. The three hosting-provider API roles — `timeweb`, `vdsina`
and `inferno` — were extracted from the `new_lab` repository, where they
lived as loose local roles under `roles/`.

### Fixed

- **inferno**: deleted the trailing `inferno | wait for vps` task in
  `detect.yml`. Its URL ended in `| map(attribute='orderid') | string`, which
  stringifies the map generator itself rather than extracting a value — the
  request went to `…&orderid=<generator object sync_do_map at 0x…>` — where
  the equivalent expression that sets `inferno_order_id` correctly ends in
  `first`. It also re-registered `vps_info`, clobbering the looped result set
  `inferno_order_id` is derived from, though harmlessly: every read of that
  result happens at an earlier task position, before this task runs (and the
  `vps | check` handler re-registers the same name for its own `until:` wait,
  same harmless clobber). The task was redundant either way.
- **inferno**: `inferno_ssh_key_id` was set from
  `inferno_ssh_keys_info.json.list | map(attribute='id') | first` — the first
  key in the account, ignoring the configured `inferno_server.ssh_key`
  entirely. On any account holding more than one key, the reinstall could
  provision the wrong key and lock the operator out of the rebuilt server. It
  now selects by name, as `timeweb` and `vdsina` do.
- **inferno**: the `vps | check` handler had `until:` with no `retries` or
  `delay`, silently taking Ansible's implicit 3 attempts at 5s — 15 seconds
  for a VPS that has just been destroyed and rebuilt. Now 30 attempts at 10s,
  giving it five minutes to come back.

- **inferno**: `config | add ssh keys` now sends `name` in its upload body, and
  the whole ssh block is guarded with `when: not inferno_ssh_key_exists`.
  Previously the upload omitted `name`, so the provider named the key itself
  and `detect.yml`'s `selectattr('name', 'equalto', inferno_server.ssh_key)`
  could never match the key the role had just uploaded — the role was
  unbootstrappable against an account that did not already hold the key under
  exactly that name. `detect.yml` now sets `inferno_ssh_key_exists` and only
  derives `inferno_ssh_key_id` when the key is present, mirroring `timeweb`;
  `config.yml` re-reads `sshkey.list` after uploading and derives the id from
  there rather than guessing at the undocumented `sshkey.add` response shape.
  The guard also stops the task re-issuing `sshkey.add` on every run, and its
  `changed_when` now defaults the missing-`message` case instead of raising.
- **vdsina**: removed `vdsina_server.password`. It was declared and never sent
  — `config | create server` posts only `datacenter`, `name`, `server-plan`,
  `ssh-key` and `template` — so a caller wiring a vaulted secret into it gained
  nothing and put the cleartext into anything that renders `vdsina_server`
  (`ansible-inventory --list`, the fact cache, task output; no task in the role
  sets `no_log`). Servers are provisioned key-only.

### Changed

- **timeweb**: internal registers and set_facts gained a `timeweb_` prefix, so
  the role passes `ansible-lint` at the `production` profile. `timeweb_account`
  and `timeweb_server` are the public interface and are unchanged, so inventory
  needs no changes. Renamed: `presets_info`, `preset_id`, `servers_info`,
  `server_exists`, `server_id`, `ssh_keys_info`, `ssh_key_exists`,
  `ssh_key_id`, `os_info` and `os_id`. The `os_id`/`preset_id` keys in the
  `POST /servers` request body are the Timeweb Cloud API's own field names and
  were left as-is.
- **timeweb**: dropped `handlers/main.yml`, which only contained `---` and was
  never notified.
- **timeweb**: `tasks/main.yml` gained `timeweb.detect` and `timeweb.config`
  tags. `detect` carries both tags, because `config.yml` reads facts
  `detect.yml` sets — running `--tags timeweb.config` alone would otherwise
  fail on undefined variables.
- **timeweb**: the README's `## Playbook` example showed a plain
  `roles: - role: timeweb`, but the role is invoked via `include_role` with
  `delegate_to: localhost` and `become: false` because it talks to the Timeweb
  API rather than the host it provisions; the example now matches. The
  `## Variables` example also showed `os_version: '12'` and
  `ssh_key: id_ed25519` against defaults of `'13'` and `automator`, and a new
  `## Facts set by this role` table documents six of the renamed facts (it
  correctly excludes the four `*_info` registers).
- **vdsina**: internal registers and set_facts gained a `vdsina_` prefix, so
  the role passes `ansible-lint` at the `production` profile. `vdsina_account`
  and `vdsina_server` are the public interface and are unchanged, so inventory
  needs no changes. Renamed: `datacenters_info`, `datacenter_id`,
  `servers_info`, `server_exists`, `server_id`, `server_groups_info`,
  `server_group_id`, `server_plans_info`, `server_plan_id`, `ssh_keys_info`,
  `ssh_key_exists`, `ssh_key_id`, `templates_info` and `template_id`. Unlike
  `timeweb`, none of these names collide with a `POST /servers` request body
  key — VDSina's own keys (`datacenter`, `server-plan`, `ssh-key`, `template`)
  are hyphenated and lexically distinct, so no key had to be left unrenamed.
- **vdsina**: dropped `handlers/main.yml`, which only contained `---` and was
  never notified.
- **vdsina**: `tasks/main.yml` gained `vdsina.detect` and `vdsina.config`
  tags. `detect` carries both tags, because `config.yml` reads facts
  `detect.yml` sets — running `--tags vdsina.config` alone would otherwise
  fail on undefined variables.
- **vdsina**: the README's `## Playbook` example showed a plain
  `roles: - role: vdsina`, but the role is invoked via `include_role` with
  `delegate_to: localhost` and `become: false` because it talks to the VDSina
  API rather than the host it provisions; the example now matches. A new
  `## Facts set by this role` table documents the eight renamed facts that are
  useful outside the role.
- **inferno**: `inferno_vps` renamed to `inferno_server`, for consistency with
  `timeweb_server` and `vdsina_server` and with the shared role shape the
  collection README documents. Its three keys (`ssh_key`, `template`,
  `reinstall`) are unchanged, and `inferno_account` is untouched. The role has
  no references outside this collection, so this is not breaking in practice.
  Two derived facts were renamed with it, to keep `inferno_vps` from surviving
  as a prefix: `inferno_vps_order_id` → `inferno_order_id` and
  `inferno_vps_reinstall` → `inferno_reinstall`.
- **inferno**: internal registers gained an `inferno_` prefix, so the role
  passes `ansible-lint` at the `production` profile. Renamed: `templates_info`,
  `orders_info`, `vps_info`, `ssh_keys_info`, `ssh_key_result` and
  `server_reinstall`. `server_reinstall` became `inferno_reinstall_result`
  rather than `inferno_server_reinstall` — the two names would not have
  collided with `inferno_reinstall` above, but `inferno_server_reinstall`
  reads like `inferno_server.reinstall`, which it is not. Note that `vps_info`
  is also a field name in this
  provider's JSON responses — `selectattr('vps_info.ip', …)` in `detect.yml`
  and the second `vps_info` in the handler's
  `inferno_vps_info.json.vps_info.online` are API field paths and are
  deliberately untouched.
- **inferno**: `tasks/main.yml` gained `inferno.detect` and `inferno.config`
  tags in place of the flat `inferno` tag. `detect` carries both tags, because
  `config.yml` reads facts `detect.yml` sets — running
  `--tags inferno.config` alone would otherwise fail on undefined variables.
  A `flush_handlers` step was also added, because unlike `timeweb` and
  `vdsina` this role has a real handler (`vps | check`) that must run before
  the play ends.
- **inferno**: two mislabelled task names fixed — `detect | add ssh keys` in
  `config.yml` is a config-phase task and is now `config | add ssh keys`,
  and `set inferno_vps_reinstall fact` had no section prefix at all and is now
  `detect | set inferno_reinstall fact`.
- **inferno**: `README.md` and the `meta/main.yml` description were
  copy-pasted from `vdsina` — the README was titled `# vdsina` and documented
  `vdsina_account` / `vdsina_server`, neither of which exists in this role.
  Both are rewritten, and `meta/main.yml` gained the `role_name` and
  `namespace` keys and the empty `dependencies` list the other two roles
  carry. The new README documents the four facts the role sets, that this
  provider authenticates with a `cid` + `key` pair rather than a bearer token,
  and that `reinstall: true` destroys and rebuilds the server.

### Known issues

- **inferno**: `tasks/detect.yml`'s os-template selection,
  `select('match', inferno_server.template) | first`, is unanchored — Jinja's
  `match` test matches at the start of the string but not the end, so a
  server configured with `template: debian12` also matches a hypothetical
  `debian123` entry in the provider's template list, and picks whichever one
  sorts first. Same "silently provision something plausible but wrong" class
  as the SSH-key defect fixed above; not fixed here because it cannot be
  exercised without calling the live API.
- **timeweb** and **vdsina**: `config | save ansible_host fact` uses
  `ansible.builtin.lineinfile` without `create: true` against
  `{{ inventory_dir }}/host_vars/{{ inventory_hostname }}.yml`, so it fails if
  that file does not already exist — which is exactly the fresh-provision
  case these roles exist for. Not fixed here because it cannot be exercised
  without actually provisioning a server.
- **inferno**: the `- name: handlers` / `flush_handlers` step in
  `tasks/main.yml` is explicit and carries no tags, so any `--tags` selection
  drops it and handler ordering falls back to the implicit end-of-play flush.
  Left as-is because every sibling role in this collection does the same;
  `tags: always` would be the fix.
