# 02: VM Template - Rocky Linux 10

This log documents the process of creating a standard, reusable Rocky Linux 10 VM template, hardened and enabled for `cloud-init`.

---

## 1. Objective

Create a base template to quickly deploy new VMs without full re-installation. The template must be secure and allow Proxmox to customize the hostname, user, and network on each clone.

---

## 2. VM Creation (Proxmox GUI)

* **OS:** Rocky 10 ISO (`rocky-10.x-x86_64-minimal.iso`)
* **System:** QEMU Agent was left **Disabled** for the initial creation.
* **Storage:** 15GB.
* **Network:** `vmbr0`

---

## 3. OS Installation & Post-Install (VM Shell)

1.  Performed a "Minimal Install" from the Rocky installer and set a temporary root password.
2.  Booted the new VM and logged in as root.
3.  Updated the system:
    ```bash
    dnf update -y
    ```
4.  Installed the QEMU Guest Agent (the software side):
    ```bash
    dnf install qemu-guest-agent -y
    ```
5.  Installed Cloud-Init:
    ```bash
    dnf install cloud-init cloud-utils-growpart -y
    ```

---

## 4. Template Preparation & Hardening (VM Shell)

1.  Ran `cloud-init clean` to remove any machine-specific data:
    ```bash
    cloud-init clean
    ```
2.  Locked the `root` account. This is a critical security step and forces `cloud-init` to use the new user/SSH key specified during the clone.
    ```bash
    passwd -l root
    ```
3.  Powered down the VM:
    ```bash
    shutdown now
    ```

---

## 5. Finalization (Proxmox GUI)

1.  **Enable QEMU Agent:**
    * Selected the VM (left-click).
    * Went to the `Options` tab.
    * Selected `QEMU Guest Agent` and clicked `Edit`.
    * **Checked the "Use QEMU Guest Agent" box.** (This is when Proxmox shows the warning, confirming the agent needs to be installed, which I already did in Step 3).
2.  **Added Cloud-Init Drive:**
    * Went to the `Hardware` tab.
    * Clicked `Add` -> `Cloud-Init Drive`.
3.  **Converted to Template:**
    * Selected the VM (left-click).
    * Clicked the **"More"** button in the top menu bar.
    * Selected **"Convert to template"**.

---

## 6. Screenshots

<img width="800" alt="imagen" src="https://github.com/user-attachments/assets/43c07647-b780-40e2-a6ac-7527b47fe5cd" />
<img width="800" alt="imagen" src="https://github.com/user-attachments/assets/67b52090-ca5b-44fc-b364-066e9b744285" />

