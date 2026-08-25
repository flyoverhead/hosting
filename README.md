# `flyoverhead.hosting`

[![Version](https://img.shields.io/badge/version-1.0.1-blue)](galaxy.yml)
[![ansible-core](https://img.shields.io/badge/ansible--core-%E2%89%A52.16-black?logo=ansible&logoColor=white)](https://docs.ansible.com/ansible-core/devel/index.html)
[![License](https://img.shields.io/badge/license-GPL--3.0--only-green)](https://www.gnu.org/licenses/gpl-3.0)
[![Providers](https://img.shields.io/badge/providers-3-orange)](#-providers)

Ansible roles that drive VPS hosting provider APIs: create, find and destroy
servers at Timeweb Cloud, VDSina and inferno.name.

These roles talk to a provider's control-plane API. They do not configure the
resulting machine — that is `flyoverhead.server` and `flyoverhead.docker`'s
job, once the server exists and is reachable over SSH.

## 🚀 Quick Start

### Requirements

- `ansible-core >=2.16`

### Installation

Installing the collection dependencies:

```bash
ansible-galaxy collection install -r requirements.yml
pip install -r requirements.txt
```

Installing the collection itself:

```bash
ansible-galaxy collection install git+https://github.com/flyoverhead/hosting.git
```

### Roles usage

Full documentation and usage examples of role `<role>` can be found in
`roles/<role>/README.md`.

These roles run against `localhost` and are invoked with `include_role` rather
than `roles:` — see [Shared role shape](#-shared-role-shape) for the snippet and
the reasoning behind it.

## 🌐 Providers

| Name | Provider | API docs |
| :--- | :--- | :--- |
| [`timeweb`](roles/timeweb/README.md) | Timeweb Cloud | <https://timeweb.cloud/api-docs> |
| [`vdsina`](roles/vdsina/README.md) | VDSina | <https://userapi.vdsina.com> |
| [`inferno`](roles/inferno/README.md) | inferno.name | n/a |

## 🏗 Shared role shape

All three roles take the same two variables: `<provider>_account` (the API
endpoint plus credentials) and `<provider>_server` (the desired server —
size, image, SSH key, and whether it should be `present` or `absent`).

`inferno` does not fit that second half: `inferno_server` has no size and no
`state` — it has `reinstall` instead — and the role has no delete path at
all, only detect/create/reinstall.

Because the role talks to the provider's API rather than to the host it
provisions, it is run against `localhost`, not against the target itself.
Invoke it with `include_role`, not `roles:`, so `apply` can pin the
delegation and privilege escalation per-invocation:

```yaml
- name: timeweb vps
  ansible.builtin.include_role:
    name: flyoverhead.hosting.timeweb
    apply:
      become: false
      delegate_to: localhost
```

`delegate_to: localhost` sends the API calls from the controller instead of
the (possibly not-yet-existing, or about-to-be-destroyed) target host.
`become: false` matters because the controller user usually cannot `sudo` to
call out to an HTTPS API, and does not need to.

## 🧪 Testing

This collection has no test harness. `flyoverhead.server` and
`flyoverhead.docker` ship a Vagrantfile because their roles configure a
throwaway VM; these roles create and destroy real, billed servers at a
hosting provider, so there is nothing to stand up locally. Changes are
verified with `ansible-lint` and review only.

## 🔐 Credentials

Every `<provider>_account.api_key` (and `inferno_account.key`) is a live
secret. It belongs in a vault-encrypted variable, never committed in
cleartext group_vars or host_vars.

## 📄 License

GPL-3.0-only.
