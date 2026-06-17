# ansible-role-proxmox

An Ansible role that performs post-install configuration for Proxmox VE. It switches apt sources from the enterprise repositories to the no-subscription repositories and optionally removes the subscription nag from the web and mobile interfaces.

## Requirements

- Target host must be running Proxmox VE 8 (Debian Bookworm) or Proxmox VE 9 (Debian Trixie).
- Ansible fact gathering must be enabled (the role uses `ansible_facts.distribution_release` to set apt suite names).
- Must run as root or with `become: true`.

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `proxmox_remove_subscription_nag` | `true` | Patch the web and mobile UI to suppress the subscription nag popup. |
| `proxmox_enable_pvetest_repo` | `false` | Add the `pvetest` beta repository. Disabled by default; enable only on non-production hosts. |
| `proxmox_disable_ha_services` | `false` | Stop and disable the HA cluster services (`pve-ha-lrm`, `pve-ha-crm`, `corosync`). Intended for single-node installs that do not need HA. |

## What the Role Does

### Repository configuration

- Disables the `pve-enterprise` repository by setting `Enabled: false` in `/etc/apt/sources.list.d/pve-enterprise.sources`.
- Enables the Proxmox VE no-subscription repository at `/etc/apt/sources.list.d/proxmox.sources`.
- Configures the Ceph repository at `/etc/apt/sources.list.d/ceph.sources` with the enterprise entry disabled and the no-subscription entry enabled (pinned to `ceph-squid`).
- Writes `/etc/apt/sources.list.d/pve-test.sources` with `Enabled` set according to `proxmox_enable_pvetest_repo`.
- Flushes the apt cache immediately after repo changes via a `meta: flush_handlers` call.

### Subscription nag removal

When `proxmox_remove_subscription_nag` is `true` (the default):

- Deploys `/usr/local/bin/pve-remove-nag.sh`, which patches `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js` to bypass the subscription check in the web UI, and appends a JavaScript block to `/usr/share/pve-yew-mobile-gui/index.html.tpl` to suppress the nag in the mobile UI. Both patches are idempotent.
- Installs `/etc/apt/apt.conf.d/no-nag-script`, a DPkg post-invoke hook that re-runs the patch script automatically after any package update that might overwrite the widget toolkit.

After the role runs, clear your browser cache or do a hard reload (Ctrl+Shift+R) for the web UI change to take effect.

### HA service management

When `proxmox_disable_ha_services` is `true`, the role stops and disables `pve-ha-lrm`, `pve-ha-crm`, and `corosync`. This reclaims system resources on standalone nodes that are not part of a cluster.

## Example Playbook

```yaml
- hosts: proxmox
  become: true
  roles:
    - role: glennbrown.proxmox
```

Single-node setup with the pvetest repo enabled and HA services disabled:

```yaml
- hosts: proxmox
  become: true
  roles:
    - role: glennbrown.proxmox
      vars:
        proxmox_enable_pvetest_repo: true
        proxmox_disable_ha_services: true
```

## License

MIT
