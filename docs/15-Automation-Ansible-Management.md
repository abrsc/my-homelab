# 15: Automation - Ansible Controller Setup & Fleet Management

This log documents the deployment of the **Ansible Controller**, the central "Control Tower" of the homelab. This instance allows for automated updates, configuration management, and mass deployment of services (like the Wazuh agents seen in Log 14).

---

## 1. Objective

Transition from manual "per-node" administration to **Infrastructure as Code (IaC)**. 
* **Centralization:** Run commands on 10+ containers simultaneously.
* **Consistency:** Ensure every LXC has the same security patches and configurations.
* **Documentation:** The playbooks themselves serve as living documentation of the infrastructure.

---

## 2. Controller Deployment

* **Host:** Dedicated LXC Container
* **OS:** Debian 12 / 13
* **IP Address:** `192.168.1.55`
* **Resources:** 1 vCPU / 512MB RAM (Ansible is lightweight as it is agentless).

---

## 3. SSH Infrastructure (The Foundation)

Ansible communicates via SSH. To make this work securely and seamlessly:

1.  **Key Generation:** A dedicated Ed25519 SSH key was generated on the controller.
2.  **Mass Injection:** I used a custom script to inject the public key into all Proxmox LXC containers (`/root/.ssh/authorized_keys`).
3.  **Manual VM Setup:** For the Docker VM (`.60`), the key was added manually to ensure persistent access.

---

## 4. Project Structure & Git Integration

I organized the work into a dedicated repository to track changes over time.

* **Directory:** `~/homelab-infra`
* **Version Control:** Git initialized with a `.gitignore` to prevent sensitive data (like private keys) from being committed.

### Inventory Management (`hosts.ini`)
The inventory separates LXCs (running as root) from VMs (running as standard users with sudo).

```ini
[lxc_hosts]
192.168.1.50  # AdGuard
192.168.1.51  # NPM Plus
192.168.1.103 # Wazuh Manager
# ...

[lxc_hosts:vars]
ansible_user=root

[vm_hosts]
192.168.1.60  # Docker Prod

[vm_hosts:vars]
ansible_user=your_user
ansible_become=true
```

---

## 5. Validation & Initial Connectivity

The first successful test was the global "ping" command, verifying that the controller can reach every node through the Proxmox firewall.

**Command:**
```bash
ansible -i hosts.ini all -m ping
```

**Result:** All nodes returned `"ping": "pong"`, confirming the SSH keys and network paths are correctly configured.

---

## 6. Future-Proofing: Logging

To maintain a professional audit trail, I enabled logging in `ansible.cfg`:

```ini
[defaults]
log_path = /var/log/ansible/ansible.log
```
*Every playbook execution (including the Wazuh deployment) is now recorded for troubleshooting and history.*

---

## 7. Screenshots
<img width="800" alt="Ansible Ping" src="https://github.com/user-attachments/assets/91cd1a34-4987-44fa-9bc0-e9413be04b87" />
<img width="800" alt="Ansible Tree" src="https://github.com/user-attachments/assets/ba6fbf77-25b2-4185-9727-884f67a97676" />
