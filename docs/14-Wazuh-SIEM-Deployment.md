# 14: SIEM - Wazuh Deployment & Real-Time Alerting

This log documents the installation of the **Wazuh SIEM**, the automated deployment of agents across the cluster using **Ansible**, and the creation of a custom **Telegram notification system** with GeoIP enrichment.

---

## 1. Objective

The goal is to centralize security logs from all VMs and LXCs, monitor file integrity, and receive real-time alerts on mobile for critical events (like SSH brute-force or root access).

---

## 2. Wazuh Manager Installation

* **Host:** Dedicated Instance (LXC/VM)
* **IP Address:** `192.168.1.103`
* **Method:** Standard installation on Debian 12.
* **External Access:** * Secured via **Nginx Proxy Manager** (HTTPS).
    * Protected by **Cloudflare Zero Trust** (Email OTP required to see the dashboard).

---

## 3. Automated Agent Deployment (Ansible)

To avoid manual installation on every container, I developed a "Modern Method" Ansible playbook. This playbook handles GPG keys correctly (fixing the `apt-key` deprecation in Debian 12/13) and supports both Debian-based and RedHat-based (Rocky Linux) systems.

### `deploy-wazuh.yml`
```yaml
---
- name: Deploy Wazuh Agent (Modern Method)
  hosts: homelab
  become: yes
  gather_facts: yes
  vars:
    wazuh_manager: "192.168.1.103"
    wazuh_gpg_url: "[https://packages.wazuh.com/key/GPG-KEY-WAZUH](https://packages.wazuh.com/key/GPG-KEY-WAZUH)"
    wazuh_repo_deb: "deb [signed-by=/usr/share/keyrings/wazuh.gpg] [https://packages.wazuh.com/4.x/apt/](https://packages.wazuh.com/4.x/apt/) stable main"
    wazuh_repo_rpm: "[https://packages.wazuh.com/4.x/yum/](https://packages.wazuh.com/4.x/yum/)"

  tasks:
    # 0. EXCLUSIONS
    - name: Skip Alpine Linux and Wazuh Manager Host
      meta: end_host
      when:
        - ansible_os_family == "Alpine" or inventory_hostname == "wazuh"

    # 1. SETUP DEBIAN / UBUNTU (Modern GPG Handling)
    - name: Install dependencies (Debian)
      apt:
        name: [gpg, curl, apt-transport-https, lsb-release]
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Convert GPG key to keyring
      shell: |
        curl -s {{ wazuh_gpg_url }} | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg --yes
      args:
        creates: /usr/share/keyrings/wazuh.gpg
      when: ansible_os_family == "Debian"

    # 2. SETUP REDHAT / ROCKY
    - name: Add Wazuh Repository (RedHat)
      yum_repository:
        name: wazuh
        description: Wazuh repository
        baseurl: "{{ wazuh_repo_rpm }}"
        gpgcheck: yes
        gpgkey: "{{ wazuh_gpg_url }}"
        enabled: yes
      when: ansible_os_family == "RedHat"

    # 3. INSTALLATION & CONFIGURATION
    - name: Install Wazuh Agent
      package:
        name: wazuh-agent
        state: present

    - name: Set Manager IP in ossec.conf
      lineinfile:
        path: /var/ossec/etc/ossec.conf
        regexp: '<address>.*</address>'
        line: "      <address>{{ wazuh_manager }}</address>"
      notify: Restart Wazuh Agent

    - name: Enable and Start Service
      service:
        name: wazuh-agent
        state: started
        enabled: yes

  handlers:
    - name: Restart Wazuh Agent
      service:
        name: wazuh-agent
        state: restarted
```

---

## 4. Custom Telegram Integration

To receive alerts on my phone, I implemented a Python integration. This script includes **GeoIP enrichment** (showing the country and flag of the attacker) and a **cooldown mechanism** to prevent notification spam.

### Features
* **GeoIP Lookup:** Uses `ip-api.com` to identify the origin of attacks.
* **Anti-Spam:** 5-minute cooldown per IP address.
* **Formatting:** Clean Markdown messages sent to a private Telegram Bot.

### `custom-telegram.py` (Snippet)
```python
#!/var/ossec/framework/python/bin/python3
import sys, json, time, requests

# CONFIGURATION (Censored)
CHAT_ID = "XXXXXXXXXX"
COOLDOWN_TIME = 300 # 5 minutes

def get_geoip_info(ip):
    # Fetches country and flag for a given IP
    # Logic: ip-api.com -> JSON -> Flag Emoji conversion
    ...

# MAIN LOGIC
# 1. Read Wazuh Alert JSON
# 2. Check Cooldown cache
# 3. Format Markdown message (Level, Agent, GeoIP, Target)
# 4. POST to Telegram Bot API
```

---

## 5. Security & Network Flow

* **Inbound (Logs):** Agents talk to the Manager on ports **1514 (TCP/UDP)** and **1515 (TCP)**.
* **Internal Firewall:** Proxmox Rule #9 allows only these ports from the internal network to the Wazuh LXC.
* **Outbound (Alerts):** The Wazuh Manager is authorized to reach Telegram APIs and GeoIP APIs via HTTPS (Port 443).

---

## 6. Screenshots
