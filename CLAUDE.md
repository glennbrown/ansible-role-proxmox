# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Role Overview

This is an Ansible role (`ansible-role-proxmox`) that configures Proxmox VE repositories on a Proxmox host. It disables the enterprise repository, enables the no-subscription repository for both PVE and Ceph (squid), and flushes the apt cache.

## Running and Testing

Lint with ansible-lint:

```bash
ansible-lint
```

Run against a host (requires inventory):

```bash
ansible-playbook -i <inventory> <playbook>.yml
```

Test a specific task file by including it in a playbook that targets a Proxmox host.

## Structure

- `tasks/main.yml` — entry point; delegates to `repos.yml`
- `tasks/repos.yml` — all substantive tasks: disabling enterprise repos, enabling no-subscription repos for PVE and Ceph
- `handlers/main.yml` — single handler (`update_apt_cache`) triggered by repo changes
- `defaults/main.yml` — currently empty, intended for overridable defaults
- `vars/main.yml` — currently empty, intended for non-overridable vars

## Key Behaviors

- The role uses `ansible_facts.distribution_release` to set the apt suite (e.g., `bookworm`). Facts must be gathered before this role runs.
- Repo files are written using the DEB822 format (`.sources` files), not the legacy one-liner `.list` format.
- Ceph is pinned to the `ceph-squid` release for both enterprise (disabled) and no-subscription (enabled) entries.
- The `flush_handlers` meta task at the end of `repos.yml` forces an immediate apt cache update rather than deferring to the end of the play.
