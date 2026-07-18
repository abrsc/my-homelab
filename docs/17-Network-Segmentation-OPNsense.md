# 17: Network Segmentation - OPNsense & VLANs

This log documents the core network segmentation implemented using **OPNsense** as a virtualized firewall/router within Proxmox.

---

## 1. Objective

The goal of this deployment was to move away from a flat /24 home network and implement enterprise-style network micro-segmentation.

* **Isolation:** Separate vulnerable lab environments from the rest of the network.
* **Security Testing:** Create a dedicated, untraceable zone for cybersecurity testing.
* **IP Management:** Implement logical VLAN numbering mapped to IP subnets.

---

## 2. VLAN Architecture

I designed the network using three distinct VLANs. For ease of management and readability, the VLAN ID matches the 3rd octet of the IP subnet. OPNsense acts as the DHCP server and default gateway for all of them.

| VLAN ID | Subnet | Name | Purpose & Status |
| :--- | :--- | :--- | :--- |
| **VLAN 20** | `10.10.20.0/24` | `WEB_SERVICES` | Dedicated zone for web-facing services and reverse proxies. *(Currently in migration phase).* |
| **VLAN 30** | `10.10.30.0/24` | `AD_LAB` | Isolated environment for the Microsoft Hybrid Identity Lab (Active Directory, Entra ID). Documented in Log 18. |
| **VLAN 40** | `10.10.40.0/24` | `CYB_LAB` | Highly restricted security zone used for Kali Linux and penetration testing. |

---

## 3. The SecOps "Tor-Only" Network (VLAN 40)

The most complex part of this setup is **VLAN 40**. Since I use this zone for cybersecurity testing, malware analysis, and Kali Linux VMs, I wanted strict guarantees about outbound traffic.

Instead of just providing standard internet access, OPNsense is configured to enforce **transparent Tor routing** for this specific subnet.

### How it works:
1. A VM is deployed and attached to VLAN 40 in Proxmox.
2. OPNsense assigns an IP via DHCP (`10.10.40.x`).
3. **Firewall / NAT Rules:** Any outbound traffic leaving this VLAN is intercepted by the firewall and forced through the Tor network before reaching the public internet.
4. **Failsafe (Kill Switch):** If the Tor service goes down, the traffic is dropped. The VMs on this VLAN are physically incapable of leaking their real public IP to the internet.

---

## 4. Current Status & Next Steps

**[✓] Implemented:**
* OPNsense VM deployment with multiple virtual NICs.
* VLAN creation and Proxmox network bridge tagging.
* DHCP server configured per VLAN.
* Transparent Tor routing for VLAN 40.

**[ ] Planned:**
* Complete the IP migration of web containers to VLAN 20.

## 5. Screenshots

<img width="800" alt="Dashboard view of OPNsense Interfaces Assignment, displaying logical separation with VLAN 20 for Web Services, VLAN 30 for the Microsoft Lab, and VLAN 40 for SecOps." src="https://github.com/user-attachments/assets/58504080-7abe-49cf-973e-3dbf21d44de0" />
<img width="800" alt="OPNsense Destination NAT rules for the CYB_LAB interface. It demonstrates transparent Tor routing by redirecting outbound DNS requests (port 53) to Tor port 9053, and external web traffic to the Tor TransPort 9040." src="https://github.com/user-attachments/assets/e1747b3a-22af-45e2-a39e-8afa076e2a75" />

