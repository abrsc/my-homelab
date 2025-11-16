# 03: LXC - AdGuard Home

This log documents the deployment of AdGuard Home as a network-wide DNS blocker in a lightweight LXC container.

---

## 1. Objective

Deploy a fast and efficient ad-blocker for the entire LAN (homelab and home devices) using minimal server resources, and configure it as the primary DNS resolver for the network.

---

## 2. LXC Creation (Proxmox GUI)

* **Template:** Debian 13 (latest stable release).
* **Storage:** 8GB
* **Network:** Set to a **Static IP: 192.168.1.50**. This is critical for the network DNS.

---

## 3. Installation (LXC Shell)

1.  Logged into the LXC shell as `root`.
2.  Installed  `curl`.
    ```bash
    apt update
    apt install curl
    ```
4.  Used the official AdGuard Home installation script:
    ```bash
    curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
    ```
---

## 4. Setup (AdGuard Web Interface)

1.  Accessed the setup wizard at `http://192.168.1.50:3000`.
2.  Created the admin user.
3.  **Upstream DNS Servers:** Configured AdGuard to use `1.1.1.1` (Primary) and `8.8.8.8` (Secondary) as its providers.
4.  **Security Filters:** Added several extra blocklists (e.g., security/malware filters) for enhanced protection.
5.  **DNS Rewrite:** Configured a DNS rewrite rule so that `adguard-home.local` resolves directly to `192.168.1.50` for easy access.

---

## 5. Router Configuration

1.  Logged into my main router.
2.  Disabled the default ISP-provided DNS servers.
3.  Set the router's primary (and only) DNS server to the AdGuard IP: **`192.168.1.50`**.
4.  This forces all devices on the network to use AdGuard for DNS resolution.

---
## 6. Screenshots
<img width="800" alt="dashboard" src="https://github.com/user-attachments/assets/a7c399ff-d81b-4ff7-8b7e-479dfb8f44eb" />
<img width="800" alt="dhcp settings" src="https://github.com/user-attachments/assets/c5567e79-39b2-4630-8094-749c1fb6c835" />
<img width="800" alt="filters" src="https://github.com/user-attachments/assets/f6b09992-5072-4346-9def-cbb044305fda" />
