# Prometheus Ansible Playbook

This repository contains a small Ansible playbook to download, extract and install Prometheus on target hosts.

## What this playbook does

- Creates a system user `prometheus` (no home, no login shell)
- Creates configuration and TSDB directories (`/etc/prometheus` and `/var/lib/prometheus`)
- Downloads the Prometheus tarball to `/home/ubuntu` using `get_url`
- Unarchives the tarball on the remote host
- Copies `prometheus` and `promtool` binaries to `/usr/local/bin`
- Copies `prometheus.yml` to `/etc/prometheus`

## Location

Playbook: `playbook.yaml`
Inventory: `inventory.ini`

Run from the `Prometheus-Playbook` directory.

## Requirements

- Ansible (2.9+ recommended)
- SSH access to the target hosts defined in `inventory.ini`
- The account you connect with must have permissions to create users, write to `/usr/local/bin` and `/etc` (use `become: true` or run with `--ask-become-pass`)

Note: The playbook currently downloads the Prometheus release tarball for version `3.7.3` (as per the URL). If you change the URL/version, the playbook dynamically finds the extracted directory using the `find` module, so the rest of the tasks will adapt.

## Key variables produced by the playbook

The playbook registers and sets a few variables you can reuse in later tasks or roles:

- `prometheus_download` — result returned by `get_url` (registered)
- `prometheus_archive` — path to the downloaded archive (set via `set_fact`)
- `prometheus_filename` — basename of the downloaded archive
- `unarchived_file` — result of the `unarchive` task (registered)
- `found_prometheus_dir` — result of the `find` task searching for `prometheus-*`
- `prometheus_dir` — final path to the extracted directory (set via `set_fact` from `found_prometheus_dir.files[0].path`)

Use `{{ prometheus_dir }}` in subsequent tasks to reference the extracted directory without hardcoding versioned names.

## How to run

Run the full playbook:

```bash
ansible-playbook -i inventory.ini playbook.yaml
```