# 01: Proxmox VE 9 Installation

This log documents the initial setup of the Proxmox VE 9 server.

## 1. BIOS Configuration

Before installation, the following BIOS settings were required:

* **CSM Support:** `Disabled` (To force UEFI boot and edit Secure Boot settings).
* **Secure Boot:** `Disabled` (Required to boot the Proxmox installer).
* **Integrated Graphics:** `Set to 64MB` (To free up system RAM).

---

## 2. Installation Process

* **Installer:** Used Rufus as Ventoy had issues with the PVE 9 ISO.
* **Target Disk:** `Crucial P310 500GB NVME`
* **Static IP:** `192.168.1.100`
* **Gateway (Router):** `192.168.1.1`
* **Hostname:** `proxmox.local`

<img width="1338" height="775" alt="imagen" src="https://github.com/user-attachments/assets/32d51705-b18d-4223-ace8-15b1922b1539" />
<img width="2552" height="1255" alt="imagen" src="https://github.com/user-attachments/assets/94fe6579-0266-4205-a0cc-68a87034953b" />

---

## 3. Post-Installation: Fixing APT Repositories (PVE 9 / Trixie)

The default PVE 9 installation points to paid enterprise repositories, which causes `401 Unauthorized` errors during `apt update`.

**Solution:** Followed the official Proxmox documentation, I used `nano` to disable the enterprise repos and add the public 'no-subscription' repos.

**1. Disabled the `pve-enterprise` repository:**
Renamed the file so it's ignored by `apt`.

```shell
mv /etc/apt/sources.list.d/pve-enterprise.sources /etc/apt/sources.list.d/pve-enterprise.sources.bak
```

**2. Created the new `proxmox.sources` file:** Used `nano` to create `/etc/apt/sources.list.d/proxmox.sources` with the following content:

```plaintext
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

**3. Edited the existing `ceph.sources` file:** Used `nano /etc/apt/sources.list.d/ceph.sources` to replace the enterprise content with the public 'no-subscription' content:

```plaintext
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/ceph-archive-keyring.gpg
```

**4. Final Update:** Ran the update and upgrade to finalize the installation.

```shell
apt update
apt full-upgrade
```

---

## 4. System Benchmark

Ran `pveperf` to check system health.

<img width="996" height="602" alt="imagen" src="https://github.com/user-attachments/assets/80dbc38c-cf5e-40d8-ac3a-48fa16b0e7bd" />
