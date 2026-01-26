# My Proxmox Homelab Project

Welcome to the documentation for my personal homelab. This project transforms a recycled PC into a fully functional virtualization server using Proxmox VE.

The goal is to build a hands-on environment to practice system administration, networking, and cybersecurity concepts for my studies in System & Network Administration (Spanish "ASIR" vocational program) and future certifications.

---

## 1. Hardware Stack
* **Server:** Custom Workstation (Asus PRIME H510M-A R2.0)
* **CPU:** Intel Core i5-10400
* **RAM:** 64GB DDR4 2666MHz
* **GPU:** NVIDIA GeForce RTX 3060 12GB (Planned for AI & Transcoding)
* **Storage:**
    * **System & ISOs:** 480GB SATA SSD
    * **VM Storage:** 1TB NVMe
* **Power:** Salicru SPS SOHO+ 850VA UPS (Managed via NUT)
* **Backup Node:** Repurposed Ryzen PC (Proxmox Backup Server)

---

## 2. Network Map & IP Allocation

The network follows a strict zoning policy to avoid conflicts between static servers and DHCP clients.

| Zone / Range | IP Address | Hostname | Role | Type |
| :--- | :--- | :--- | :--- | :--- |
| **Network Gear** | `.1` | `ZTE:F6640` | Router / Gateway | Hardware |
| *(.1 - .49)* | | | | |
| **Core Infra** | `.50` | `adguard-home-dns` | DNS Resolver | LXC |
| *(.50 - .89)* | `.51` | `npmplus` | Reverse Proxy | LXC |
| | `.52` | `cloudflared` | Cloudflare Tunnel | LXC |
| | `.53` | `tailscale-gateway` | Subnet Router | LXC |
| | `.54` | `apache-guacamole` | Remote Access GW | LXC |
| | `.55` | `ansible-controller` | Automation / IaC | LXC |
| | `.60` | `docker-prod` | Docker Host | VM |
| **Physical Hosts** | `.99` | `proxmox-pbs` | Backup Server | PBS 4.1 |
| *(.90 - .101)* | `.100` | `proxmox-ve` | Hypervisor | PVE 9.1 |
| | `.101` | `LEGION-ARNO` | Admin Workstation | Windows |
| **Applications** | `.102` | `wordpress` | Web Server | LXC |
| *(.102 - .129)* | `.103` | `wazuh` | SIEM / Security | LXC |
| | `.120` | `ghostfolio` | Finance Tracker | LXC |
| **Lab / AD Zone** | `.130` | `win-server-dc` | Domain Controller (AD) | VM (Planned) |
| *(.130 - .199)* | `.143` | `win-ltsc-insti` | Daily Driver / Work | VM |
| **DHCP Clients** | `.200 - .254` | - | Phones, PCs, IoT | Dynamic |

---

## 3. Software Stack

* **Hypervisor:** Proxmox VE 9
* **Boot USB:** Ventoy & Rufus
* **Network & Security:** Cloudflare Zero Trust (Tunnel), Nginx Proxy Manager, Tailscale (VPN/Subnet Router).
* **Deployed Services:**
    * **AdGuard Home** (DNS/AdBlock)
    * **WordPress** (Production CMS behind Cloudflare Tunnel)
    * **Uptime Kuma** (Monitoring via Docker)
* **Planned:** Active Directory, OPNsense, Kubernetes (K3s).
---

## 4. Project Documentation

All detailed installation, configuration, and troubleshooting steps are logged in the `/docs` folder.

* **Phase 1: Installation**
    * **[📄 01: Proxmox VE Installation](./docs/01-Installation-PVE9.md)**
    * *(This log covers BIOS settings, PVE installation, and fixing APT repositories)*
    * **[📄 02: VM Template - Rocky Linux 10](./docs/02-Template-Rocky10.md)**
    * *(Steps for creating a reusable, cloud-init-enabled Rocky 10 template)*
    * **[📄 06: Storage Expansion - Backup Strategy](./docs/06-Storage-Expansion-4TB.md)**
    * *(Adding a dedicated backup drive, XFS formatting, and directory mounting)*
    * **[📄 08: Infrastructure Migration](./docs/08-Infrastructure-Migration.md)**
    * *(Documenting the move to new hardware and repurposing the old node as PBS)*
    * **[📄 09: Power Protection - UPS & NUT](./docs/09-UPS-Configuration-NUT.md)**
    * *(Automated graceful shutdown configuration)*
    * **[📄 10: Backup Server Setup](./docs/10-PBS-Setup.md)**
    * *(Installation and configuration of the dedicated PBS node)*

* **Phase 2: Core Services**
    * **[📄 03: LXC - AdGuard Home](./docs/03-LXC-AdGuard.md)**
    * *(Fast deployment of a network-wide DNS ad-blocker)*
    * **[📄 04: LXC - Nginx Proxy Manager Plus](./docs/04-LXC-NginxProxyManager.md)**
    * *(Internal reverse proxy for clean URLs and local SSL)*
    * **[📄 05: LXC - WordPress & Cloudflare Tunnel](./docs/05-LXC-WordPress-Cloudflare.md)**
    * *(Production deployment via Community Scripts, secured behind Cloudflare Zero Trust)*
    * **[📄 07: Docker Host - Uptime Kuma](./docs/07-Docker-UptimeKuma.md)**
    * *(Monitoring stack deployed via Docker Compose on Rocky Linux)*
    * **[📄 12: VM - Cloud Storage Aggregation](./docs/12-Storage-OneDrive-Aggregation.md)**
    * *(Merging multiple OneDrive accounts into a 4TB+ virtual drive using Rclone Union & Docker)*

* **Phase 3: Networking & Security**
    * **[📄 11: LXC - Tailscale Subnet Router](./docs/11-LXC-Tailscale-SubnetRouter.md)**
    * *(Secure remote access gateway, Subnet Routing, and global AdBlocking)*
    * **[📄 13: Security Architecture - Zero Trust & Firewall](./docs/13-Security-Architecture.md)**
    * *(Implementation of Network Micro-segmentation, standard Security Groups, and "Default Deny" policies across the cluster)*
