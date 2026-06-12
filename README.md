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

## What the Role Does

### Repository configuration

- Disables the `pve-enterprise` repository by setting `Enabled: false` in `/etc/apt/sources.list.d/pve-enterprise.sources`.
- Enables the Proxmox VE no-subscription repository at `/etc/apt/sources.list.d/proxmox.sources`.
- Configures the Ceph repository at `/etc/apt/sources.list.d/ceph.sources` with the enterprise entry disabled and the no-subscription entry enabled (pinned to `ceph-squid`).
- Flushes the apt cache immediately after repo changes via a `meta: flush_handlers` call.

### Subscription nag removal

When `proxmox_remove_subscription_nag` is `true` (the default):

- Deploys `/usr/local/bin/pve-remove-nag.sh`, which patches `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js` to bypass the subscription check in the web UI, and appends a JavaScript block to `/usr/share/pve-yew-mobile-gui/index.html.tpl` to suppress the nag in the mobile UI. Both patches are idempotent.
- Installs `/etc/apt/apt.conf.d/no-nag-script`, a DPkg post-invoke hook that re-runs the patch script automatically after any package update that might overwrite the widget toolkit.

After the role runs, clear your browser cache or do a hard reload (Ctrl+Shift+R) for the web UI change to take effect.

## Example Playbook

```yaml
- hosts: proxmox
  become: true
  roles:
    - role: glennbrown.proxmox
```

To skip the subscription nag removal:

```yaml
- hosts: proxmox
  become: true
  roles:
    - role: glennbrown.proxmox
      vars:
        proxmox_remove_subscription_nag: false
```

## License

MIT
