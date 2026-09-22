# Ubuntu Home Server / VPS Bootstrapper

A production-grade, hardened Ansible playbook designed to automatically configure, secure, and containerize a freshly installed **Ubuntu Server** node. 

This infrastructure-as-code automation handles core OS updates, migrates SSH away from generic brute-force vectors using native **systemd socket overrides**, establishes a tight **UFW firewall perimeter**, handles administrative user creation, installs **Docker**, and deploys an automated **Nginx Proxy Manager** gateway to handle free, automatically renewing Let's Encrypt TLS certificates.

## 🚀 Key Features
* **System Upgrades & Maintenance:** Automatically refreshes `apt` caches, upgrades core packages safely, and installs default management tooling (`ufw`, `fail2ban`, `htop`).
* **Modern Systemd SSH Isolation:** Seamlessly relocates the public SSH connection vector off standard port 22 to a customizable entry point (`2222`) utilizing modern Ubuntu systemd socket overrides rather than old-school file-edits.
* **Hardened Perimeter Security:** Locks incoming network footprints down completely with `UFW`, opening explicit tracks *only* for public web traffic (`80`, `443`) and your custom remote SSH shell configuration.
* **Native Docker Integration:** Bootstraps the official Docker runtime engine alongside its native APT-managed python bindings (`python3-docker`) to guarantee clean, non-destructive container automation.
* **Isolated Reverse Proxy & TLS:** Deploys an automated Nginx Proxy Manager gateway that requests and automatically handles hands-free 90-day updates for free Let's Encrypt SSL/TLS certificates.

---

## 📂 Project Isolation & Setup

### 1. Isolate the Project via Sparse-Checkout
To pull this specific tool out of your repository workspace without cluttering your system with your complete monorepo setup, open your terminal and run:

```bash
# Initialize an empty local directory
mkdir Home_Server && cd Home_Server
git init

# Link your multi-project workspace as the remote engine
git remote add origin https://github.com/sergio-a-juarez-1/Ansible.git

# Enable sparse-checkout and pull the target server directory
git sparse-checkout set Home_Server
git pull origin main
```

### 2. File Structure
Ensure your working directory matches this structure inside `Home_Server/`:
```text
.
├── inventory.ini
├── site.yml
└── README.md
```

---

## ⚡ Setup & Execution

### Prerequisites
Make sure your local automation control machine has access to the community distribution extensions required for firewall and container management:
```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install community.docker
```

### Phase 1: First-Time Bootstrap Run (Port 22 Setup)
On your very first execution pass against a clean OS, ensure your `inventory.ini` points to your initial setup user on port 22. Pass the `-k` and `-K` flags to manually feed the initial SSH and sudo credentials:

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini site.yml -k -K
```

### Phase 2: Ongoing Production Run (Post-Hardening)
Once the playbook executes successfully, your default SSH listener shifts to port `2222` and your admin user `sysadmin` is created. Update your `inventory.ini` to map to Phase 2 tracking, and run subsequent maintenance smoothly:

```bash
ansible-playbook -i inventory.ini site.yml
```

---

## 🔒 Automated TLS Certificate Setup & Admin UI Access

To maximize your home server's defense posture, **the Nginx Proxy Manager Admin Dashboard (Port 81) is intentionally kept closed on the public firewall** and bound strictly to the local loopback interface (`127.0.0.1`). 

### 1. Establish an Encrypted SSH Management Tunnel
To access your configuration manager securely from your local laptop, open an encrypted local port forwarding tunnel from your terminal:

```bash
ssh -L 8080:127.0.0.1:81 sysadmin@YOUR_SERVER_IP -p 2222
```

### 2. Configure Your Routes & Certificates
Keep that terminal session running in the background, open your local web browser, and navigate to: **`http://localhost:8080`**

1. Authenticate using default fallback administration credentials:
   * **Email:** `admin@example.com`
   * **Password:** `changeme`
2. Update your administrator profile parameters immediately to establish secure unique values.
3. Navigate to **Hosts** ➔ **Proxy Hosts** ➔ **Add Proxy Host**.
4. Set your domain target configurations, click the **SSL** block, select **Request a New SSL Certificate**, toggle **Force SSL**, and agree to the Let's Encrypt terms. 

*Nginx Proxy Manager will silently trigger an automated internal check every 24 hours. If any certificate is within 30 days of expiration, it will automatically handle renewal communication with Let's Encrypt over public port 80 dynamically.*
