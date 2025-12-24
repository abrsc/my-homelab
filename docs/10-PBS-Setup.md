# 10: Dedicated Backup Server (PBS) Setup

This log documents the repurposing of the legacy Proxmox host into a dedicated **Proxmox Backup Server (PBS)**. This ensures that backups are physically separated from the production hypervisor, following the "3-2-1 Backup Strategy" principles.

---

## 1. Hardware Reallocation

The "Old Node" (Ryzen 5 2400G) was decommissioned as a hypervisor and repurposed.
* **CPU:** AMD Ryzen 5 2400G
* **RAM:** 8GB DDR4 (Stable stick retained, faulty stick removed)
* **OS Drive:** 500GB NVMe
* **Storage Drive:** 4TB Seagate Ironwolf (HDD)
* **Role:** Dedicated Backup Target

---

## 2. Installation & Network Configuration

* **OS:** Proxmox Backup Server 4.1 (Debian 12 Bookworm base)
* **Hostname:** `proxmox-pbs.lan`
* **IP Address:** `192.168.1.99/24` (Static)
* **Gateway:** `192.168.1.1`
* **BIOS Settings:**
    * *Restore on AC Power Loss:* **Power On** (Ensures server reboots after outage).
    * *Wake-on-LAN:* **Enabled**.

---

## 3. Storage Architecture (The 4TB Drive)

Given the hardware constraints (8GB RAM), I opted **against ZFS** for the backup storage to avoid memory exhaustion.

### Configuration
* **Type:** Directory (Ext4/XFS backend)
* **Filesystem:** XFS (Chosen for reliability with large files).
* **Datastore Name:** `backup-storage`
* **Path:** `/mnt/datastore/backup-storage`

This setup is lightweight and allows full utilization of the disk without the RAM overhead of ZFS Deduplication features, while still benefiting from PBS's client-side deduplication.

---

## 4. Post-Install Operations

1.  **Repositories:** Configured "No-Subscription" repositories for updates.
2.  **Updates:** Full system upgrade performed (`apt full-upgrade`) to ensure latest security patches and kernel.
3.  **Boot Check:** System rebooted successfully without requiring kernel parameters updates.

## 5. Integration

The server was linked to the main Proxmox VE node via the "Datacenter > Storage" menu using the secure Fingerprint exchange.
