# Ansible: Zero to Automation Hero

Welcome to the official repository for the **Ansible Automation** training course! This repository houses all the configuration files, infrastructure-as-code manifests, playbooks, roles, and lab exercises covered throughout the comprehensive 32-section automation curriculum.

---

## 📊 Course Overview
* **Total Scope:** 32 Sections • 111 Lectures • 4h 8m total runtime
* **Core Goal:** Master configuration management, multi-node deployment, cloud integration (AWS), and production-grade security.
* **Format:** Theoretical architecture breakdowns paired with sandbox environment challenges (VirtualBox/AWS).

---

## 📂 Repository Structure

```text
├── 01-environment-setup/  # VirtualBox, Ubuntu VM setup, and Linux/Mac installation scripts
├── 02-yaml-syntax/         # Core YAML files, syntax guides, and formatting challenges
├── 03-inventories/        # Static INI & YAML inventory layouts across distinct environments
├── 04-playbooks/          # Core modules (Service, Command, Debug) and Handlers
├── 05-variables-loops/    # Variable registers, arrays, external var files, and loop constructs
├── 06-ansible-roles/      # Modular codebase design, directory structures, and Ansible Galaxy
├── 07-cloud-dynamic/      # AWS integration, dynamic inventory parsing (ec2.py), and workflows
├── 08-security-vault/     # Secure secrets management via Ansible Vault and HashiCorp Vault
├── 09-iac-integrations/   # Orchestration pipelines using Terraform and custom Python modules
├── challenges/            # Hands-on labs (Web deployment, Roles architecture, YAML challenges)
└── practical-hacks/       # Advanced error handling blocks, velocity hacks, and Tower overviews
```

---

## 🛠️ Getting Started

### Prerequisites
* A Linux or macOS environment (or a Windows machine capable of running VirtualBox)
* A text editor of choice (e.g., VS Code)
* An active AWS Free Tier account (for the cloud infrastructure modules)

### Installation & Test Play
```bash
# 1. Clone this configuration management repository
git clone https://github.com

# 2. Navigate into the course directory
cd ansible-course-labs

# 3. Verify your local Ansible controller installation
ansible --version

# 4. Test execution against a local sandbox module
ansible-playbook -i 03-inventories/hosts.yaml 04-playbooks/first-playbook.yaml
```

---

## 📘 Detailed Syllabus Breakdown

### 🖥️ Local Sandboxing & Fundamentals
* **Lab Provisioning:** Spinning up Ubuntu target environments via VirtualBox and installing the Ansible core engine across macOS and Linux.
* **YAML Mechanics:** Deep-dive into syntax fundamentals, list formatting, and key-value mapping validation.
* **Inventory Control:** Building nested host groupings under both classic INI styles and contemporary declarative YAML architectures.
* **⚡ Inventory Challenge:** Building cross-environment mappings isolating production and staging tiers.

### 📜 Playbooks, Variables & Modular Roles
* **Task Execution:** Crafting declarative playbooks leveraging the `Service` engine and managing task flow states via state `Handlers`.
* **State Verification:** Increasing debugging verbosity outputs and registering array variations during system loops.
* **Modular Engineering:** Restructuring monolithic execution paths into clean, self-contained `Ansible Roles` and tracking custom structural dependencies.
* **⚡ Web & Role Challenges:** Executing a complete multi-tier web server stack using reusable configuration components.

### ☁️ Cloud scale, Dynamic Ecosystems & Security
* **Dynamic Infrastructure:** Utilizing dynamic lookup tools (`ec2.py`) to query active cloud resources across AWS instances in real time.
* **Secrets Hardening:** Encrypting high-sensitivity production strings locally using `Ansible Vault`, alongside native enterprise integrations with `HashiCorp Vault`.
* **Orchestration Pipelines:** Intersecting infrastructure provisioning tools (`Terraform`) directly into downstream configuration operations.
* **Advanced Debugging:** Building sophisticated error containment wrappers using defensive execution `Blocks` and exploring commercial `Ansible Tower` scaling mechanisms.
