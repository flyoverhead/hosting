# Changelog

All notable changes to `flyoverhead.hosting`.

## 1.0.0

Initial release. The three hosting-provider API roles — `timeweb`, `vdsina`
and `inferno` — were extracted from the `new_lab` repository, where they
lived as loose local roles under `roles/`.

### Fixed

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
