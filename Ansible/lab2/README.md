# Lab 07 — Apache role and custom webpage (Ansible Lab 2)

This folder demonstrates a simple Ansible role-based setup that deploys an Apache web server and a static webpage using a role named `apache` (see `roles/apache`). The playbook `playbook.yaml` in this directory applies that role to the target hosts.

I used the environment/preparation steps from Lab 1 (`Ansible/lab1/README.md`) as the baseline for the control node and managed hosts. Make sure you have completed the Lab 1 environment setup (SSH keys, `ansible` user, `ansible.cfg` or inventory available) before running this lab.

## Contents

- `playbook.yaml` — top-level playbook that applies the `apache` role to `hosts: all`.
- `roles/apache/` — role that contains tasks, handlers, defaults, templates and tests for deploying Apache and a demo webpage.

## How to run

1. From the control node, ensure you are in this directory:

```bash
cd Ansible/lab2
```

2. (Optional) Create a local `ansible.cfg` if you did not in lab1. Example is available in `Ansible/lab1/README.md`.

If you need to (re)create the `apache` role skeleton locally (for development), run:

```bash
ansible-galaxy init apache
```

3. Run the playbook:

```bash
ansible-playbook -i ../lab1/inventory.ini playbook.yaml
```

Notes:

- The command above re-uses the `inventory.ini` from Lab 1; you may instead point to another inventory that lists your target hosts.
- If your control node is set up with `ansible.cfg` pointing to `inventory.ini`, you can run simply `ansible-playbook playbook.yaml`.

## What the role does (summary)

- Installs Apache (package manager used depends on the target OS).
- Deploys a demo index page (from role `templates/` or `files/`).
- Starts and enables the Apache service.
- Provides handlers to restart/reload Apache when needed.

Inspect `roles/apache/tasks/main.yml` and `roles/apache/templates/` to see the exact files and variables used.
