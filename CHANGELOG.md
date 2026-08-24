# Changelog

All notable changes to `flyoverhead.hosting`.

## 1.0.0

Initial release. The three hosting-provider API roles — `timeweb`, `vdsina`
and `inferno` — were extracted from the `new_lab` repository, where they
lived as loose local roles under `roles/`.

### Fixed

- **inferno**: deleted the trailing `inferno | wait for vps` task in
  `detect.yml`. It re-registered `vps_info`, destroying the looped result set
  that `inferno_order_id` is derived from, and its URL ended in
  `| map(attribute='orderid') | string`, which stringifies the map generator
  itself — the request went to `…&orderid=<generator object sync_do_map at
  0x…>`. The equivalent expression that sets `inferno_order_id` correctly ends
  in `first`. The task was redundant anyway: the `vps | check` handler already
  performs this wait properly, with `until:`.
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
  tags. `detect` carries both tags, because `configure.yml` reads facts
  `detect.yml` sets — running `--tags timeweb.config` alone would otherwise
  fail on undefined variables.
- **timeweb**: the README's `## Playbook` example showed a plain
  `roles: - role: timeweb`, but the role is invoked via `include_role` with
  `delegate_to: localhost` and `become: false` because it talks to the Timeweb
  API rather than the host it provisions; the example now matches. The
  `## Variables` example also showed `os_version: '12'` and
  `ssh_key: id_ed25519` against defaults of `'13'` and `automator`, and a new
  `## Facts set by this role` table documents the ten renamed facts.
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
  tags. `detect` carries both tags, because `configure.yml` reads facts
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
  rather than `inferno_server_reinstall`, which would have collided with
  `inferno_reinstall` above. Note that `vps_info` is also a field name in this
  provider's JSON responses — `selectattr('vps_info.ip', …)` in `detect.yml`
  and the second `vps_info` in the handler's
  `inferno_vps_info.json.vps_info.online` are API field paths and are
  deliberately untouched.
- **inferno**: `tasks/main.yml` gained `inferno.detect` and `inferno.config`
  tags in place of the flat `inferno` tag. `detect` carries both tags, because
  `configure.yml` reads facts `detect.yml` sets — running
  `--tags inferno.config` alone would otherwise fail on undefined variables.
  A `flush_handlers` step was also added, because unlike `timeweb` and
  `vdsina` this role has a real handler (`vps | check`) that must run before
  the play ends.
- **inferno**: two mislabelled task names fixed — `detect | add ssh keys` in
  `configure.yml` is a config-phase task and is now `config | add ssh keys`,
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
