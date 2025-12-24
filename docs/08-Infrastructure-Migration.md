# 08: Major Infrastructure Migration & Upgrade

This log documents the migration from the legacy Ryzen-based host to a new, more powerful Intel-based workstation, and the repurposing of old hardware into a dedicated Backup Server.

---

## 1. Motivation & Trigger Event

The migration was a strategic decision driven by hardware changes and future learning goals:

1.  **Hardware Constraint:** The previous node (Ryzen 5 2400G) experienced a DIMM module failure, dropping usable memory from 16GB to 8GB. While the system was stable, 8GB creates an immediate ceiling for virtualization.
2.  **Proactive Scaling:** To avoid halting my learning progress (specifically for memory-heavy labs like Kubernetes or Corporate Network simulation), I decided to upgrade immediately rather than waiting.
3.  **Platform Upgrade:** Transitioning to a newer Intel Core i5 architecture with **64GB of RAM** ensures the homelab is future-proof for years to come.

---

## 2. The Migration Strategy (Backup & Restore)

Instead of cloning the disk (which would carry over incompatible drivers), I opted for a clean install:

1.  **Backup:** All LXCs and VMs were backed up to `.vma.zst` files on the external backup drive.
2.  **Transfer:** The backup drive was physically moved to the new host.
3.  **Restore:** Used Proxmox "Restore" feature, mapping the old storage configurations to the new high-speed `local-nvme` thin pool.

---

## 3. Storage Architecture Changes

To maximize performance on the new hardware, I separated the OS from the VM data:

* **OS Drive (SATA SSD):** dedicated to Proxmox system, ISOs, and Backup files.
* **VM Drive (NVMe):** dedicated to running VM disks.

### Technical Step: Reclaiming Space on Boot Drive
By default, Proxmox creates a `local-lvm` partition on the boot drive. Since VMs are now on NVMe, this space was reclaimed for the root filesystem:

1.  **Remove the unused volume:**
    ```bash
    lvremove /dev/pve/data
    ```
2.  **Resize the root volume:**
    ```bash
    lvresize -l +100%FREE /dev/pve/root
    ```
3.  **Expand the filesystem:**
    ```bash
    resize2fs /dev/mapper/pve-root
    ```

---

## 4. Repurposing Old Hardware (PBS)
The old Ryzen hardware was not discarded. After removing the faulty RAM module, the machine is being repurposed as a dedicated **Proxmox Backup Server (PBS)** with 8GB of stable RAM.
