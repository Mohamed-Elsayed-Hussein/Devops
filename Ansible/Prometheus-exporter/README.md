# Prometheus Node Exporter Ansible Playbook

This repository contains a small Ansible playbook to download, install, and run Prometheus Node Exporter as a systemd service on target hosts.

## Repository layout

- `playbook.yaml` — Installs and configures Node Exporter
- `inventory.ini` — Sample inventory targeting a host with the `ubuntu` user and an SSH key
- `.ansible.cfg` — Defaults: remote_user=ubuntu, become enabled, inventory default path, etc.
- `commands/` — Placeholder for helper scripts or commands (not required to run the playbook)

## What this playbook does

- Creates a dedicated system user `node_exporter` (no home, no login shell, UID 1050)
- Downloads Node Exporter v1.10.2 linux-amd64 tarball to `/home/ubuntu`
- Unarchives the tarball on the remote host
- Dynamically locates the extracted directory (pattern: `node_exporter-*`) and stores its path in a fact
- Copies the `node_exporter` binary to `/usr/local/bin/`, owned by `node_exporter`
- Creates a systemd unit file at `/etc/systemd/system/node_exporter.service`
- Reloads systemd, then enables and starts the `node_exporter` service

Notes:
- The playbook uses `gather_facts: false` and relies on `.ansible.cfg` for privilege escalation (`become: true`).
- SELinux-related tasks are present but commented out; see the SELinux note below if needed.

## Requirements

- Ansible (Core 2.12+ or newer recommended)
- SSH access to the target hosts
- The connecting user must have sudo privileges (configured via `.ansible.cfg`)
- Target system using systemd

## Defaults and assumptions

- Default remote user: `ubuntu` (from `.ansible.cfg`)
- Privilege escalation: enabled (become: true)
- Download destination in tasks: `/home/ubuntu` (matches the default remote user)
- Service port: 9100 (default Node Exporter)

If your remote user is not `ubuntu`, either:
- Override the user in `inventory.ini` via `ansible_user=<user>` and update the download path in the playbook from `/home/ubuntu` to `/home/<user>`, or
- Override at runtime: `-u <user>` and also update the download path accordingly.

## Variables and facts set by the playbook

These are useful for reuse or extension:

- `node_exporter_download` — Result returned by `get_url` (registered)
- `node_exporter_archive` — Full path to the downloaded archive (set via `set_fact`)
- `node_exporter_filename` — Basename of the downloaded archive
- `found_node_exporter_dir` — Result of the `find` task searching for `node_exporter-*`
- `node_exporter_dir` — Path to the extracted directory (from `found_node_exporter_dir.files[0].path`)

Use `{{ node_exporter_dir }}` in subsequent tasks to reference the dynamically discovered extract directory.

## How to run

Using the provided inventory file:

```bash
ansible-playbook -i inventory.ini playbook.yaml
```

Additional examples:
- Increase verbosity: `ansible-playbook -i inventory.ini playbook.yaml -vvv`
- Override SSH key if not specified in inventory: `ansible-playbook -i inventory.ini playbook.yaml --private-key /path/to/key.pem`
- Override remote user: `ansible-playbook -i inventory.ini playbook.yaml -u ubuntu`

Note: `.ansible.cfg` sets a default inventory at `/etc/ansible/hosts`. Supplying `-i inventory.ini` will override it and use the file in this repository.

## Changing the Node Exporter version

The version is controlled by the hardcoded URL in the `get_url` task in `playbook.yaml`:

```
https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
```

To upgrade:
- Update the version in the URL (both path segment and filename).
- The playbook will still work without other changes because it dynamically discovers the extracted directory using the `find` task with `node_exporter-*` pattern.

## Verifying installation

- Check service status on the target host:

```bash
sudo systemctl status node_exporter
```

- Verify metrics endpoint (on the target host):

```bash
curl http://localhost:9100/metrics
```

If accessing remotely, ensure port 9100 is allowed by your firewall/security groups.

## SELinux (optional)

If SELinux is enforcing on the target, you may need to label the binary and restore context. The playbook contains commented tasks showing an example using `semanage` and `restorecon`. To use them:

1. Ensure the SELinux tools are installed (e.g., `policycoreutils-python-utils` on newer Debian/Ubuntu or `policycoreutils-python` on older, `policycoreutils` on RHEL/CentOS).
2. Uncomment the SELinux tasks in `playbook.yaml` and rerun the playbook.

## Troubleshooting

- Permission denied when writing to `/usr/local/bin` or `/etc/systemd/system`:
  - Ensure privilege escalation is active. `.ansible.cfg` enables `become: true`; do not disable it.
- Different remote username/home directory:
  - Update the download path in the playbook (it currently uses `/home/ubuntu`).
- Service not starting:
  - Run `journalctl -u node_exporter -e` on the target host for logs.
- Port 9100 not reachable remotely:
  - Open firewall/security group for TCP 9100 or tunnel to the host.

## Uninstall (manual)

On the target host:

```bash
sudo systemctl stop node_exporter
sudo systemctl disable node_exporter
sudo rm -f /etc/systemd/system/node_exporter.service
sudo rm -f /usr/local/bin/node_exporter
sudo userdel node_exporter || true
sudo systemctl daemon-reload
```

## Inventory example

`inventory.ini` in this repo demonstrates a single host with SSH key auth:

```
54.196.173.129  ansible_user=ubuntu ansible_ssh_private_key_file=~/Downloads/pyh.pem
```

Adjust IP, user, and key path to match your environment.
