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

## 4. Security & Encryption

To protect data against physical theft of the drive or server, **Client-Side Encryption** was enforced.

* **Method:** AES-256 GCM.
* **Key Management:** A master encryption key was generated on the PVE Node.
* **Storage:** The `.enc` key file is securely stored in Bitwarden and off-site Cloud Storage.
* **Result:** Data leaving the PVE node is encrypted *before* transmission. The PBS server only stores encrypted chunks.

---

## 5. Backup Strategy (GFS & Integrity)

A robust **Grandfather-Father-Son (GFS)** retention policy was configured in the Backup Job to balance history vs. disk space.

### 5.1 Retention Policy
* **Keep Last:** `3` (Keeps the very last 3 runs, useful if multiple manual backups occur same day)
* **Keep Daily:** `7` (Ensures exactly one backup per day is kept for the last week)
* **Keep Weekly:** `4` (One backup per week for the last month)
* **Keep Monthly:** `12` (One backup per month for the last year)
* **Keep Yearly:** `1` (Long-term archival)

### 5.2 Data Integrity (Verify Job)
To detect "bit rot" or disk corruption, a scheduled verification task runs on the PBS server.
* **Schedule:** Every Sunday at 10:00.
* **Policy:** Re-verify snapshots older than 30 days.

---

## 6. Power Protection (NUT Client)

Since the UPS is connected to the PVE node, this server is configured as a network **Slave**.

* **Mode:** `netclient`
* **Monitor:** Listens to `ups@192.168.1.100`.
* **Behavior:** When the Master (PVE) reports "Low Battery", this server shuts down gracefully to prevent filesystem corruption.

---

## 7. Post-Install Operations

1.  **Repositories:** Configured "No-Subscription" repositories for updates.
2.  **Updates:** Full system upgrade performed (`apt full-upgrade`).
3.  **Boot Check:** System rebooted successfully (Video output issues on Ryzen 2400G resolved by headless operation/kernel update).

## 8. Screenshots
<img width="800" alt="PBS SUMMARY" src="https://github.com/user-attachments/assets/98406959-0ee2-4683-bd55-9934ef1ce1f0" />
<img width="800" alt="PBS CONTENT" src="https://github.com/user-attachments/assets/92bf71e5-cd52-4189-af91-d9411d1963cb" />
<img width="800" alt="PVE BACKUP JOB RETENTION" src="https://github.com/user-attachments/assets/a29ea1d1-9001-402e-a955-7caed5415567" />


