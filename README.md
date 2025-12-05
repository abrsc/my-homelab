# My Proxmox Homelab Project

Welcome to the documentation for my personal homelab. This project transforms a recycled PC into a fully functional virtualization server using Proxmox VE.

The goal is to build a hands-on environment to practice system administration, networking, and cybersecurity concepts for my studies in System & Network Administration (Spanish "ASIR" vocational program) and future certifications.

---

## 1. Hardware Stack

* **Server:** Recycled PC (Gigabyte A320M-S2H V2)
* **CPU:** AMD Ryzen 5 2400G
* **RAM:** 16GB DDR4
* **Storage:** 500GB NVMe SSD
* **Network:** 5-Port Gigabit Switch

---

## 2. Software Stack

* **Hypervisor:** Proxmox VE 9
* **Boot USB:** Ventoy & Rufus
* **Network & Security:** Cloudflare Zero Trust (Tunnel), Nginx Proxy Manager.
* **Deployed Services:**
    * **AdGuard Home** (DNS/AdBlock)
    * **WordPress** (Production CMS behind Cloudflare Tunnel)
* **Planned:** Active Directory, OPNsense, Uptime Kuma, Tailscale.
---

## 3. Project Documentation

All detailed installation, configuration, and troubleshooting steps are logged in the `/docs` folder.

* **Phase 1: Installation**
    * **[📄 01: Proxmox VE 9 Installation](./docs/01-Installation-PVE9.md)**
    * *(This log covers BIOS settings, PVE installation, and fixing APT repositories)*
    * **[📄 02: VM Template - Rocky Linux 10](./docs/02-Template-Rocky10.md)**
    * *(Steps for creating a reusable, cloud-init-enabled Rocky 10 template)*
    * **[📄 06: Storage Expansion - 4TB HDD](./docs/06-Storage-Expansion-4TB.md)**
    * *(Adding a dedicated backup drive, XFS formatting, and directory mounting)*

* **Phase 2: Core Services**
    * **[📄 03: LXC - AdGuard Home](./docs/03-LXC-AdGuard.md)**
    * *(Fast deployment of a network-wide DNS ad-blocker)*
    * **[📄 04: LXC - Nginx Proxy Manager Plus](./docs/04-LXC-NginxProxyManager.md)**
    * *(Internal reverse proxy for clean URLs and local SSL)*
    * **[📄 05: LXC - WordPress & Cloudflare Tunnel](./docs/05-LXC-WordPress-Cloudflare.md)**
    * *(Production deployment via Community Scripts, secured behind Cloudflare Zero Trust with custom PHP/Nginx tuning)*

* **Phase 3: Networking & Security**
    * *[Coming Soon]*
