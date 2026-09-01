# Ansible Configuration Management Infrastructure

This repository contains **Ansible playbooks and roles** designed to automate infrastructure provisioning, system configuration management, and application deployments across multi-environment fleets.

## 🚀 Key Features

* **Automated System Hardening:** Secures OS baselines, manages firewalls, and enforces SSH security protocols.
* **Idempotent Deployments:** Ensures safe, repeatable executions without disrupting active production services.
* **Dynamic Inventories:** Integrates natively with cloud providers to automatically discover and group target hosts.
* **Modular Architecture:** Utilizes reusable roles and collections to keep configurations clean and scalable.

## 📂 Repository Structure

```text
├── group_vars/           # Global and group-specific variables
├── host_vars/            # Host-specific variable overrides
├── inventories/          # Environment definitions (staging, production)
├── roles/                # Reusable, modular automation tasks
│   ├── common/           # Base system utilities and packages
│   └── webserver/        # Nginx/Apache configuration and setup
├── site.yml              # Master playbook orchestrating all roles
└── ansible.cfg           # Global Ansible behavior configurations
```

## 🛠️ Prerequisites

Before running the playbooks, ensure your local controller meets these requirements:

1. **Ansible:** version 2.15+ installed locally
2. **Python:** version 3.10+ with required library dependencies
3. **SSH Access:** Valid SSH keys configured for passwordless target node authentication

## 💻 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com
cd your-ansible-repo
```

### 2. Verify Host Connectivity
Run an ad-hoc ping command to ensure you can reach the staging infrastructure:
```bash
ansible all -m ping -i inventories/staging
```

### 3. Execute the Playbook
Run the master configuration playbook against your target environment:
```bash
ansible-playbook site.yml -i inventories/staging
```

## 🔒 Security & Vault Policy

Sensitive data (such as API keys, passwords, and private certificates) must **never** be stored in plaintext. Always encrypt secret files using **Ansible Vault**:

```bash
# Encrypt a sensitive file
ansible-vault encrypt group_vars/all/vault.yml

# Execute a playbook using encrypted data
ansible-playbook site.yml -i inventories/production --ask-vault-pass
```
