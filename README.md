# 🧩 Ansible Playbooks Collection

This repository contains a curated set of **Ansible playbooks** organized by system type and purpose.  
Each category targets specific infrastructure components used in a homelab or production environment — such as Ubuntu 
servers, Proxmox hosts, Docker services, and Checkmk monitoring.

---

## 📁 Repository Structure

```
Ansible Playbooks/
├── .gitignore
├── README.md
│
├── common/
│   ├── config-add-sshkey.yaml
│   ├── maint-diskspace.yaml
│   ├── maint-reboot-required.yaml
│   └── maint-reboot.yaml
│
├── docker/
│   ├── docker-certs-enable.yaml
│   ├── docker-certs.yaml
│   ├── inst-docker-ubuntu.yaml
│   └── maint-docker-clean.yaml
│
├── notifications/
│   ├── notify-discord.yaml
│   └── notify-telegram.yaml
│
├── openwrt/
│   └── upd-opkg.yaml
│
├── proxmox/
│   └── inst-vm-core.yaml
│
└── ubuntu/
    ├── inst-zsh.yaml
    └── upd-upgrade-apt.yaml
    
````
## ✅ Structure Summary
| Folder                               | Description                                                                                                                     |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **`common/`**                        | General-purpose system maintenance playbooks — add SSH keys, check disk space, and handle system reboots.                       |
| **`docker/`**                        | Docker-related playbooks — install Docker, configure TLS certificates, enable secure daemon access, and clean up unused images. |
| **`notifications/`**                 | Notification integrations — send alerts via Discord or Telegram for system events or playbook results.                          |
| **`openwrt/`**                       | Playbooks for managing and updating OpenWrt routers (e.g., package updates).                                                    |
| **`proxmox/`**                       | Tasks related to Proxmox virtual machines — install essential VM agents and monitoring tools.                                   |
| **`ubuntu/`**                        | Ubuntu host setup and maintenance — install core packages like Zsh and perform system updates/upgrades.                         |
| **Root (`README.md`, `.gitignore`)** | Project documentation and Git configuration files.                                                                              |

---

## ⚙️ Usage

### 1️⃣ Run a playbook manually
You can run any playbook using the standard Ansible command:

```bash
ansible-playbook -i inventory.ini "Ansible Playbooks/ubuntu/upd-apt.yaml"
````

You can limit execution to a specific group or host:

```bash
ansible-playbook -i inventory.ini "Ansible Playbooks/ubuntu/maint-reboot.yaml -l vms"
```

---

## 🧠 Variables

Some playbooks use dynamic targeting through a variable called `target_group`:

```yaml
hosts: "{{ target_group | d('all') }}"
```

If not specified, the playbook defaults to all hosts in your inventory.

To run only on a specific group:

```bash
ansible-playbook "Ansible Playbooks/ubuntu/upd-apt.yaml" -e "target_group=UbuntuServers"
```

---

## 💬 Telegram Notifications

Certain playbooks (such as `maint-diskspace.yaml` and `maint-reboot-required.yaml`) are configured to send **Telegram alerts** for important events, like:

* Disk space exceeding a defined threshold
* Systems requiring a reboot after updates

These use the Telegram Bot API via Ansible’s `uri` module.

### 🔐 Setup Instructions

#### 1. Create a Telegram Bot

1. Open Telegram and start a chat with **[@BotFather](https://t.me/BotFather)**
2. Send the following commands:

   ```
   /start
   /newbot
   ```
3. Follow the prompts to name your bot and get your **bot token**, which looks like:

   ```
   123456789:ABCDefGhIjKlmNoPQRstuVWxyz
   ```

#### 2. Get your chat ID

1. Send a message (e.g. “hello”) to your new bot in Telegram.

2. Open this URL in your browser (replace `<TOKEN>` with your bot token):

   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```

3. Copy the `"chat":{"id":<YOUR_CHAT_ID>}` value from the JSON response.

#### 3. Store as environment variables (or in Semaphore secrets)

To keep credentials safe, set them as environment variables:

```bash
export TELEGRAM_BOT_TOKEN=123456789:ABCDefGhIjKlmNoPQRstuVWxyz
export TELEGRAM_CHAT_ID=987654321
```

Or, if you use Semaphore, create a **secret group**:

```yaml
TELEGRAM_BOT_TOKEN: 123456789:ABCDefGhIjKlmNoPQRstuVWxyz
TELEGRAM_CHAT_ID: 987654321
```

Then attach that secret to your Semaphore template.

#### 4. Verify by sending a test message

```bash
curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
     -d chat_id=$TELEGRAM_CHAT_ID \
     -d text="✅ Telegram notifications configured successfully!"
```

---

## 🚀 Examples

### Check disk usage and alert if >80%

```bash
ansible-playbook "Ansible Playbooks/ubuntu/upd-apt.yaml"
```

### Check for pending reboots and send notification

```bash
ansible-playbook "Ansible Playbooks/ubuntu/maint-reboot-required.yaml"
```

### Install QEMU Guest Agent on all VMs

```bash
ansible-playbook "Ansible Playbooks/proxmox/inst-qemu-agent.yaml" -e "target_group=VMs"
```

---

## 🧱 Dependencies

* [Ansible](https://www.ansible.com/) ≥ 2.14
* SSH access to target systems
* Supported OS types: Ubuntu, Debian, Proxmox, Docker hosts

Optional:

* [Semaphore](https://github.com/ansible-semaphore/semaphore) for web-based automation
* Telegram bot (for alerting playbooks)

---

## 📬 Contributing

You can extend the collection by adding new playbooks to the relevant category folder.
Follow existing naming conventions and ensure each playbook:

* Uses `target_group` for host targeting
* Includes comments and clear task names
* Is tested on the latest Ansible version

---

## 🏁 License

This repository is licensed under the **MIT License**.
You’re free to use and adapt it for personal or organizational use.

---

### 🧠 Maintainer Notes

* Keep secrets (like API keys) stored in Semaphore secret groups or environment variables.
* Use `gather_facts: no` on minimal systems like OpenWrt to avoid Python dependency errors.
* Always test updates on a small set of hosts before running playbooks globally.


