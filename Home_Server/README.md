# Ubuntu Home Server / VPS Bootstrapper

A production-grade Ansible playbook designed to automatically configure, harden, and containerize a freshly installed **Ubuntu Server** node. 

This infrastructure-as-code automation performs OS updating, migrates SSH away from generic brute-force sweeps using native **systemd socket overrides**, establishes a tight **UFW firewall perimeter**, handles administrative deployment users, installs **Docker**, and pushes real-time build completions directly to chat webhooks.

## 🚀 Key Features
* **System Upgrades & Tooling:** Automatically refreshes `apt` package trees, upgrades core packages, and sets up system maintenance suites (`ufw`, `fail2ban`, `htop`).
* **Modern Systemd SSH Isolation:** Moves your public SSH connection path off port 22 to a customizable entry point (`2222`) utilizing native modern Ubuntu systemd socket overrides instead of old-school `sshd_config` line-edits.
* **Firewall Isolation:** Locks incoming network footprints down completely with `UFW`, opening explicit tracks only for web dashboards (`80`, `443`) and your remote SSH shell configuration.
* **Dockerized Readiness:** Bootstraps the official stable Docker container engine runtime and spins up an unprivileged custom deployment user account (`sysadmin`) with passwordless `sudo` infrastructure.
* **Verification Smoke Test:** Automatically spins up a live container engine deployment running an Alpine-based Nginx landing web dashboard to instantly verify external access paths.

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

## 🛠️ Configuration Files

### 1. `inventory.ini`
This file maps your target host networks, authentication usernames, and system tracking ports:
```ini
[my_servers]
192.168.122.66 ansible_user=u ansible_port=2222
```

### 2. `site.yml`
This is your master automated build configuration sequence:
```yaml
---
- name: Boostrap and Secure New Linux Server with Docker and Alerts
  hosts: all
  become: true
  vars:
    # Change these variables to suit your needs
    created_user: "sysadmin"
    ssh_port: "2222"
    # Provide a Discord/Slack webhook URL to test notifications (optional)
    notification_webhook: "https://discord.com"
    
  tasks:
    - name: 1. SYSTEM MAINTENANCE & UPDATES
      block:
        - name: Ensure package cache is updated and upgrade OS packages
          ansible.builtin.apt:
            update_cache: true
            upgrade: dist
            autoremove: true
          when: ansible_os_family == "Debian"

        - name: Install essential system tools
          ansible.builtin.package:
            name:
              - curl
              - git
              - ufw
              - htop
              - fail2ban
            state: present

    - name: 2. SECURITY HARDENING (UFW & SSH SYSTEMD SOCKET)
      block:
        - name: Create systemd override directory for SSH socket
          ansible.builtin.file:
            path: /etc/systemd/system/ssh.socket.d
            state: directory
            mode: '0755'

        - name: Configure custom port in systemd SSH socket drop-in
          ansible.builtin.copy:
            dest: /etc/systemd/system/ssh.socket.d/listen.conf
            content: |
              [Socket]
              ListenStream=
              ListenStream={{ ssh_port }}
            mode: '0644'
          notify: Reload systemd and restart ssh socket

        - name: Configure UFW defaults (deny incoming, allow outgoing)
          community.general.ufw:
            direction: "{{ item.direction }}"
            policy: "{{ item.policy }}"
          loop:
            - { direction: 'incoming', policy: 'deny' }
            - { direction: 'outgoing', policy: 'allow' }

        - name: Allow SSH on custom port via UFW
          community.general.ufw:
            rule: allow
            port: "{{ ssh_port }}"
            proto: tcp

        - name: Allow standard web traffic
          community.general.ufw:
            rule: allow
            port: "{{ item }}"
            proto: tcp
          loop: ['80', '443']

        - name: Enable UFW firewall
          community.general.ufw:
            state: enabled

    - name: 3. USER MANAGEMENT
      block:
        - name: Create a sudo deployment user
          ansible.builtin.user:
            name: "{{ created_user }}"
            shell: /bin/bash
            groups: sudo
            append: true

        - name: Allow passwordless sudo for the new user
          ansible.builtin.lineinfile:
            path: /etc/sudoers.d/90-cloud-init-users
            line: "{{ created_user }} ALL=(ALL) NOPASSWD:ALL"
            validate: '/usr/sbin/visudo -cf %s'
            create: true
            mode: '0440'

    - name: 4. DOCKER INSTALLATION & SETUP
      block:
        - name: Install Docker via convenience script
          ansible.builtin.shell: |
            curl -fsSL https://docker.com | sh
          args:
            creates: /usr/bin/docker

        - name: Start and enable Docker service
          ansible.builtin.service:
            name: docker
            state: started
            enabled: true

        - name: Add user to docker group
          ansible.builtin.user:
            name: "{{ created_user }}"
            groups: docker
            append: true

    - name: 5. DEPLOY A COOL CONTAINER (Nginx Welcome Dashboard)
      block:
        - name: Run an Nginx container to verify deployment
          community.docker.docker_container:
            name: web_dashboard
            image: nginx:alpine
            state: started
            restart_policy: always
            published_ports:
              - "80:80"

    - name: 6. CHOREO & NOTIFICATIONS
      block:
        - name: Send a completion alert webhook
          ansible.builtin.uri:
            url: "{{ notification_webhook }}"
            method: POST
            body_format: json
            body:
              content: "🚀 **Ansible Success!** Server `{{ inventory_hostname }}` has been fully hardened, updated, and Dockerized."
            status_code:
          ignore_errors: true
          when: notification_webhook != "" and "api/webhooks" in notification_webhook

  handlers:
    - name: Reload systemd and restart ssh socket
      ansible.builtin.systemd:
        daemon_reload: true
        name: ssh.socket
        state: restarted
```

---

## ⚡ Setup & Execution

### Prerequisites
Make sure your local automation control machine has access to the community distribution extensions required for firewall and container management:
```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install community.docker
```

### Run the Playbook
Because a blank server initialization starts without validated local cryptographic host fingerprints, bypass standard host verification checking parameters on your very first bootstrapping execution pass:

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini site.yml -k -K
```

* **`-k`**: Prompts manually for your remote target SSH system authentication user password.
* **`-K`**: Prompts for your administrative privilege escalation `sudo` password string.

---

## ⚠️ Known Quirks & Edge Cases

### 1. Modern Ubuntu Systemd Socket Management
Modern versions of Ubuntu manage SSH bindings using isolated **systemd socket activation templates** (`ssh.socket`) rather than standard `sshd_config` profiles. 

If your deployment runs into an intermediate connectivity loss or timeout during the transition phases, log straight into your target server window terminal session and execute a forced configuration array processing flush manually:
```bash
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
```

### 2. Hypervisor Network Routing Bounds
If you are deploying this orchestration loop to local virtualized environments (like internal KVM/Libvirt bridges), your host system may route traffic down distinct hypervisor lines that register as **routed traffic** inside the Linux routing layer. 

If your connections return a `Connection refused` error despite open firewall profiles, verify your routing state or add your virtual host network mask directly to your server whitelist settings:
```bash
sudo ufw route allow proto tcp to any port 2222
sudo ufw allow from 192.168.122.0/24
sudo ufw reload
```
