# 11: LXC - Tailscale Subnet Router & Remote Access

This log documents the deployment of **Tailscale** using Proxmox Community Scripts.
This container acts as a **Subnet Router**, allowing secure, zero-config VPN access to the entire local network (`192.168.1.x`) and forcing remote devices to use the local AdGuard DNS.

---

## 1. Objective

* **Remote Access:** Access Proxmox, SSH, and internal VMs securely from outside (4G/5G).
* **Zero Trust:** No open ports (8006/22) on the public router.
* **AdBlocking Everywhere:** Force mobile devices to use the home AdGuard instance for DNS resolution, even when away.
* **Subnet Routing:** Use a single lightweight container to route traffic to the whole LAN.

---

## 2. Installation (Community Script)

I used the standard Proxmox VE Community Script to automate the configuration of the LXC container.

* **Command:**
    ```bash
    bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/addon/add-tailscale-lxc.sh)"
    ```
* **Configuration:**
    * **OS:** Debian 13..
    * **Resources:** 1 vCPU, 512MB RAM.
    * **Network:** DHCP.
    * **Hostname:** `tailscale-gateway`

* **Authentication:**
    Post-installation, I rebooted the LXC to ensure all settings were applied. I then ran `tailscale up` and authenticated using the generated configuration link.

---

## 3. Subnet Router Configuration

To allow this single container to route traffic to the rest of the Proxmox network (Proxmox host, PBS, VMs), I configured it as a **Subnet Router**.

### 3.1 CLI Configuration (LXC Console)
Advertised the local subnet (`192.168.1.0/24`) to the Tailnet:

```bash
tailscale up --advertise-routes=192.168.1.0/24 --accept-routes
```

### 3.2 Admin Console Approval (Web UI)
By default, advertised routes are disabled for security.
1.  Accessed the **Tailscale Admin Console**.
2.  Located the `tailscale-gateway` machine.
3.  **Edit Route Settings** -> Enabled the toggle for `192.168.1.0/24`.

---

## 4. Critical Fix: LXC IP Forwarding

Out of the box, the LXC container dropped routed packets because Linux IP forwarding is disabled by default. This resulted in successful connections to the gateway, but timeouts when trying to reach other LAN devices.

**The Fix:**
I had to manually enable IPv4 forwarding in the container's kernel settings.

1.  **Verify status:** `cat /proc/sys/net/ipv4/ip_forward` (Returned `0`).
2.  **Enable permanently:**
    Created a configuration file `/etc/sysctl.d/99-tailscale.conf` inside the LXC:
    ```bash
    echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
    echo 'net.ipv6.conf.all.forwarding = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
    sysctl -p /etc/sysctl.d/99-tailscale.conf
    ```
3.  **Result:** Instant connectivity to `192.168.1.100` (Proxmox) from a remote 4G connection.

---

## 5. DNS Configuration (Global AdBlock)

To block ads on mobile devices while traveling, I configured Tailscale to use my internal AdGuard Home instance.

1.  **Tailscale Admin Console** -> **DNS**.
2.  **Global Nameservers:** Added Custom Nameserver `192.168.1.50` (AdGuard LXC IP).
3.  **Override Local DNS:** `Enabled` (Forces all traffic to use AdGuard).

---

## 6. Security Hardening

To prevent unauthorized access if the Google account is compromised:
* **Device Approval:** Enabled "Require Device Approval" in Tailscale settings. New devices cannot join the network until manually approved by me in the admin panel.
* **Key Expiry:** Disabled for the gateway, but keys for other devices expires.

---

## 7. Screenshots

<img width="800" alt="Tailscale Admin Panel" src="https://github.com/user-attachments/assets/4c86cbee-f554-45c4-958d-03bdda682ec6" />
<img width="800" alt="Ping" src="https://github.com/user-attachments/assets/58b695e2-0385-4eac-93d8-6570378716ed" />

