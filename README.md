# Infrastructure-Ansible

Ansible project for home lab automation. Used as an AWX project source.

## Structure

```
├── ansible.cfg                         # Ansible configuration
├── requirements.yml                    # Collection dependencies (auto-installed by AWX)
├── inventories/
│   └── production/
│       └── hosts.yml                   # TrueNAS + home lab inventory
├── playbooks/
│   ├── overview.yml                    # Full TrueNAS system overview
│   ├── query_truenas_info.yml          # System info, pools, shares
│   ├── truenas_current_state.yml       # Documented state snapshot
│   ├── configure_truenas.yml           # General configuration tasks
│   ├── manage_truenas_apps.yml         # App lifecycle management
│   ├── manage_smb_shares_actual.yml    # SMB share management (live config)
│   ├── manage_smb_shares.yml           # SMB share management (template)
│   ├── manage_virtualization.yml       # LXC container/VM lifecycle
│   ├── discover_container_customizations.yml  # Discovery helper (local output)
│   └── sync_azerothcore_configs.yml    # AzerothCore env templates
├── group_vars/
│   └── truenas/
│       ├── apps.yml                    # Documented app inventory
│       └── containers.yml             # Documented container inventory
└── roles/                              # Reusable roles (empty)
```

## AWX Setup

### Credential: TrueNAS API Key

Create a **Custom Credential Type** in AWX to inject the API key:

**Input configuration (YAML):**

```yaml
fields:
  - id: truenas_api_key
    type: string
    label: TrueNAS API Key
    secret: true
required:
  - truenas_api_key
```

**Injector configuration (YAML):**

```yaml
env:
  TRUENAS_API_KEY: "{{ truenas_api_key }}"
```

Then create a credential of that type with your TrueNAS API key value and attach it to any job template that targets TrueNAS.

### Project

- **SCM Type:** Git
- **SCM URL:** `<this repo's URL>`
- **SCM Branch:** `main`
- **Update on Launch:** enabled (recommended)

### Inventory

Use **Sourced from a Project** and point to `inventories/production/hosts.yml`.

### Execution Environment

The default `awx-ee` image should have `community.general` available, but verify before running. AWX will auto-install collections from `requirements.yml` if you enable **Update on Launch** on the project.

## Hosts

| Host         | Connection    | Purpose                                    |
| ------------ | ------------- | ------------------------------------------ |
| `truenas-01` | `local` (API) | TrueNAS Scale — media, storage, containers |

## Notes

- All TrueNAS playbooks use `ansible_connection: local` and call the TrueNAS REST API via `uri` tasks with `delegate_to: localhost`. No SSH required.
- `discover_container_customizations.yml` and `sync_azerothcore_configs.yml` write output to the local machine. When run from AWX, set `local_output_dir` / `config_dest_dir` via **Extra Variables** to a writable path (e.g., `/tmp/output`).
