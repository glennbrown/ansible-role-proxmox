# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Role Overview

This is an Ansible role (`ansible-role-proxmox`) that configures Proxmox VE repositories on a Proxmox host. It disables the enterprise repository, enables the no-subscription repository for both PVE and Ceph (squid), and flushes the apt cache.

## Running and Testing

Lint:

```bash
ansible-lint
```

## Structure

- `tasks/main.yml` — entry point; delegates to `repos.yml` and conditionally `nag.yml`
- `tasks/repos.yml` — disabling enterprise repos, enabling no-subscription repos for PVE and Ceph
- `tasks/nag.yml` — deploys the nag removal script and APT hook
- `files/pve-remove-nag.sh` — the nag patch script deployed to `/usr/local/bin/`
- `handlers/main.yml` — `update_apt_cache` (triggered by repo changes) and `run_nag_removal` (triggered when the script is deployed or updated)
- `defaults/main.yml` — `proxmox_remove_subscription_nag: true`
- `vars/main.yml` — currently empty

## Key Behaviors

- The role uses `ansible_facts.distribution_release` to set the apt suite (e.g., `bookworm`). Facts must be gathered before this role runs.
- Repo files are written using the DEB822 format (`.sources` files), not the legacy one-liner `.list` format.
- Ceph is pinned to the `ceph-squid` release for both enterprise (disabled) and no-subscription (enabled) entries.
- The `flush_handlers` meta task at the end of `repos.yml` forces an immediate apt cache update rather than deferring to the end of the play.
- The nag removal script patches `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js` (web UI) and appends a JS block to `/usr/share/pve-yew-mobile-gui/index.html.tpl` (mobile UI). Both patches are idempotent via grep checks. Set `proxmox_remove_subscription_nag: false` to skip this entirely.
- `/etc/apt/apt.conf.d/no-nag-script` installs a DPkg post-invoke hook so the patch is reapplied automatically after any package update that touches the widget toolkit.
