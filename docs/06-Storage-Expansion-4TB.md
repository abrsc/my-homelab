# 06: Storage Expansion - 4TB HDD & Automated Backups

This log documents the installation and configuration of a new 4TB Seagate IronWolf HDD to serve as the primary backup destination for the Proxmox cluster.

---

## 1. Objective

Expand the server's storage capacity to implement a robust **3-2-1 backup strategy**.
* **Primary Storage:** 500GB NVMe (OS + VMs/CTs).
* **Backup Storage:** 4TB HDD (VZDump Archives, ISOs).
* **Goal:** Automate daily snapshots for all critical services without impacting the NVMe's lifespan.

---

## 2. Hardware Installation

* **Disk Model:** Seagate IronWolf 4TB (CMR, 5400 RPM).
* **Interface:** SATA III.
* **Process:**
    1.  Performed a graceful shutdown of all VMs and the Proxmox node.
    2.  Physically installed the drive in the chassis.
    3.  Verified detection in BIOS and Proxmox (`Disks` tab).

---

## 3. Disk Initialization (Proxmox GUI)

Since this disk is dedicated to backups and large files, I chose **XFS** over ZFS or LVM-Thin to maximize available space and simplicity.

1.  **Wipe Disk:** Cleared any existing partitions via `Datacenter > pve > Disks`.
2.  **Initialize with GPT:** Created a new GPT partition table.
3.  **Create Directory:**
    * **ID:** `backup-drive`
    * **Filesystem:** `xfs`
    * **Content:** `VZDump backup file`, `ISO image`.
    * **Mount Point:** `/mnt/pve/backup-drive`

---

## 4. Backup Job Configuration

Configured an automated backup job via `Datacenter > Backup` to run nightly.

### 4.1 Job Settings
* **Schedule:** Daily at 03:00 AM.
* **Selection Mode:** `All` (To automatically include any future VMs).
* **Mode:** `Snapshot` (Allows backups while VMs are running).
* **Compression:** `ZSTD` (Fast and efficient).

### 4.2 Retention Policy (Prune)
To prevent the disk from filling up, I applied a standard rotation policy:
* **Keep Last:** `7` (One week of daily history).
* **Keep Weekly:** `4` (One month of history).
* **Keep Monthly:** `2` (Two months of archive).

---

## 5. Verification

1.  Manually triggered the job ("Run now") to verify throughput.
2.  Confirmed that `.zst` archive files were created in `/mnt/pve/backup-drive/dump`.
3.  Verified that the NVMe storage is no longer used for local backups.

---

## 6. Screenshots

<img width="800" alt="Disk Summary" src="https://github.com/user-attachments/assets/29bc2075-93b9-4a03-9b29-b146d277536a" />
<img width="800" alt="Backup Config" src="https://github.com/user-attachments/assets/ec6fa363-85f5-4a09-acbc-2f4f6ff5d187" />
<img width="800" alt="Sucessful Backup" src="https://github.com/user-attachments/assets/ab3adb26-146f-42b1-9a4c-98d48997adde" />




