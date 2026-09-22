# Ubuntu Home Server / VPS Bootstrapper

A production-grade Ansible playbook designed to automatically configure, harden, and containerize a freshly installed **Ubuntu Server** node. 

This infrastructure-as-code automation performs OS updating, migrates SSH away from generic brute-force sweeps using native **systemd socket overrides**, establishes a tight **UFW firewall perimeter**, handles administrative deployment users, installs **Docker**, and deploys an automated **Nginx Proxy Manager** gateway to provision free, automatically renewing Let's Encrypt TLS certificates.

## 🚀 Key Features
* **System Upgrades & Tooling:** Automatically refreshes `apt` package trees, upgrades core packages, and sets up system maintenance suites (`ufw`, `fail2ban`, `htop`).
* **Modern Systemd SSH Isolation:** Moves your public SSH connection path off port 22 to a customizable entry point (`2222`) utilizing native modern Ubuntu systemd socket overrides instead of old-school `sshd_config` line-edits.
* **Firewall Perimeter:** Locks incoming network footprints down completely with `UFW`, opening explicit tracks only for web dashboards (`80`, `443`), proxy administration (`81`), and your remote SSH shell configuration.
* **Dockerized Engine:** Bootstraps the official stable Docker container engine runtime along with its matching Python SDK dependencies to natively handle pipeline control interfaces.
* **Automated Reverse Proxy & TLS:** Deploys an automated, visual reverse-proxy interface (Nginx Proxy Manager) that automatically requests and handles hands-free 90-day renewal updates for free Let's Encrypt SSL/TLS certs.

---

## 📂 Project Isolation & Setup

### 1. Isolate the Project via Sparse-Checkout
To pull this specific tool out of your repository workspace without cluttering your system with your complete monorepo setup, open your terminal and run:

```bash
# Initialize an empty local directory
mkdir Home_Server && cd Home_Server
git init

# Link your multi-project workspace as the remote engine
git remote add origin https://github.com

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

## 🛠️ Configuration Files

### 1. `inventory.ini`
This file maps your target host networks, authentication usernames, and system tracking ports. On your **first run**, keep your default system port configuration tracking target parameters (`22`):
```ini
[my_servers]
192.168.122.66 ansible_user=u
```

---

## ⚡ Setup & Execution

### Prerequisites
Make sure your local automation control machine has access to the community distribution extensions required for firewall and container management:
```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install community.docker
```

### First-Time Execution (Port 22 bootstrap)
Because a blank server initialization starts without validated local cryptographic host fingerprints, bypass standard host verification checking parameters on your very first bootstrapping execution pass:

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini site.yml -k -K
```

* **`-k`**: Prompts manually for your remote target SSH system authentication user password.
* **`-K`**: Prompts for your administrative privilege escalation `sudo` password string.

### Ongoing Execution (Post-Hardening Run)
Once the playbook executes successfully, your SSH listener transitions to port `2222`. Update your `inventory.ini` to look like this:

```ini
[my_servers]
192.168.122.66 ansible_user=sysadmin ansible_port=2222
```

Then you can run subsequent standard updates cleanly without password prompts (assuming your SSH key is added):
```bash
ansible-playbook -i inventory.ini site.yml
```

---

## 🔒 Automated TLS Certificate Setup

Once your playbook completes successfully, your reverse proxy infrastructure is fully operational.

1. Open your web browser and connect to: `http://YOUR_SERVER_IP:81`
2. Authenticate using default administration credentials:
   * **Email:** `admin@example.com`
   * **Password:** `changeme`
3. Update your administrator profile parameters to establish secure unique values.
4. Navigate to **Hosts** ➔ **Proxy Hosts** ➔ **Add Proxy Host**.
5. Set your domain target configurations, click the **SSL** block, select **Request a New SSL Certificate**, toggle **Force SSL**, and agree to the Let's Encrypt terms. 

*Nginx Proxy Manager will silently trigger an automated cron check every 24 hours. If any certificate is within 30 days of expiration, it will automatically handle renewal communication with Let's Encrypt over port 80 dynamically.*
