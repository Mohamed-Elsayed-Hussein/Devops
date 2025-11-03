# Lab 06 — Ansible AD-HOC Commands and Playbook for Package Management

---

## 1. Overview

This lab covers basic Ansible AD-HOC commands and a playbook for package management (installing/removing packages, starting services, and checking versions). The example playbook is `playbook.yaml` in this folder and the inventory file is `inventory.ini`.

Goals:

- Practice AD-HOC commands (ping, package, command).
- Write an idempotent playbook that uses variables and loops.
- Manage services and collect command output.

## 2. Quick AD-HOC examples

Run these from the master (control) node where Ansible is installed. Adjust `-i` path if you run the command from another directory.

- Check connectivity to all hosts:

```bash
ansible all -i inventory.ini -m ping
```

- Install `nginx` on all hosts:

```bash
ansible all -i inventory.ini -m package -a "name=nginx state=present"
```

- Verify `nginx` binary exists (example using `command` module):

```bash
ansible all -i inventory.ini -m command -a "cmd='nginx -v'"  # may output to stderr
```

- Remove `nginx` from all hosts:

```bash
ansible all -i inventory.ini -m package -a "name=nginx state=absent"
```

## 3. Playbook: package management (`playbook.yaml`)

The playbook demonstrates using variables and a loop to manage a main package and a list of tools. To run it:

```bash
ansible-playbook -i inventory.ini playbook.yaml
```

Notes about this playbook in the repository:

- The playbook defines `package_name` and `tools` variables.
- It installs packages and starts the service for `package_name`.
- The playbook registers the output of a command that checks the package version and prints it with `debug`.

Tip: The control node will display registered output; if the package prints version to stderr (some binaries do), check `output.stderr` in the playbook debug.

## 4. Environment setup (step-by-step)

These steps show how to prepare both worker nodes (hosts) and the control (master) node. Use the exact commands on the indicated machines.

### On each worker node (ansible-1, ansible-2, ansible-3)

1. Create the `ansible` user and allow passwordless sudo for automation:

```bash
sudo useradd -m ansible
# set a password interactively if needed
sudo passwd ansible

# On RHEL/CentOS use 'wheel', on Debian/Ubuntu use 'sudo' group
sudo usermod -aG wheel ansible   # RHEL
# sudo usermod -aG sudo ansible  # Debian/Ubuntu

echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
sudo chmod 440 /etc/sudoers.d/ansible
```

2. Ensure Python is available (Ansible requires Python on managed nodes):

```bash
python3 --version || sudo apt-get install -y python3  # Debian/Ubuntu
python3 --version || sudo yum install -y python3      # RHEL/CentOS
```

3. Enable SSH access for the `ansible` user (copy the control node public key):

```bash
# Run this from the control node; replace IP/hostnames as needed
ssh-copy-id -i ~/.ssh/ansible-keys.pub ansible@ansible-1
ssh-copy-id -i ~/.ssh/ansible-keys.pub ansible@ansible-2
ssh-copy-id -i ~/.ssh/ansible-keys.pub ansible@ansible-3
```

### On the control (master) node

1. Generate an SSH key pair dedicated for Ansible (if you don't have one):

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible-keys -N ''
```

2. Copy public key to worker nodes (see commands above).

3. Install Ansible on the control node (choose the command matching your distro):

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install -y ansible

# RHEL / CentOS
sudo dnf install -y ansible   # or 'yum install -y ansible' on older systems
```

4. Verify Ansible can connect to hosts:

```bash
ansible all -i inventory.ini -m ping
```

### Recommended `ansible.cfg` (local to this folder or global at `/etc/ansible/ansible.cfg`)

Create a file `ansible.cfg` in this directory with the following minimal content to simplify running commands locally:

```ini
[defaults]
inventory = inventory.ini
remote_user = ansible
host_key_checking = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

Place this file next to `inventory.ini` so you can run `ansible` and `ansible-playbook` without repeating `-i inventory.ini`.

## 5. Inventory file

`inventory.ini` in this folder contains the host groups used by the playbook. Update hostnames or add IP addresses as needed.

Example:

```ini
[WEB]
ansible-1  # optionally replace with ansible-1 ansible_host=192.168.122.6
ansible-2

[DB]
ansible-3
```

Tip: You can annotate hosts with `ansible_host=IP` and other variables per-host if your hostnames are not resolvable.

## 6. Verification & debugging

- Confirm SSH connectivity and Python:

```bash
ansible all -m ping
ansible all -a "python3 --version"
```

- Run the playbook with increased verbosity to see details:

```bash
ansible-playbook -i inventory.ini playbook.yaml -vvv
```

- If a package command prints version to stderr (common for some tools), the playbook can register `output.stderr` — check playbook debug tasks to view the right field.
