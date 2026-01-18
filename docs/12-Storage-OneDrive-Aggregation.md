# 12: VM - "Infinite" Cloud Storage (Rclone Aggregation) & File Server

This log documents the creation of a massive virtual storage array by aggregating multiple OneDrive accounts into a single logical volume using **Rclone Union**, and exposing it via **FileBrowser** and **Samba** (Dockerized).

---

## 1. Objective

To create a centralized, high-capacity file server (5TB+) without purchasing physical hard drives, leveraging free cloud storage tiers.

* **Aggregation:** Combine multiple separate OneDrive accounts (1TB+ each) into a single mount point (`/mnt/cloud_storage`).
* **Cost Efficiency:** $0 hardware cost for storage expansion.
* **Accessibility:** Files must be accessible via SMB (Windows/Mac) and Web (FileBrowser) for local users.
* **Challenges:** Overcome Docker FUSE permission issues and manage Microsoft API rate limits.

---

## 2. Architecture: The "Cloud RAID"

Instead of a traditional RAID array, I used **Rclone** with the `union` backend. This functions similarly to JBOD (Just a Bunch of Disks) but for Cloud APIs.

* **Upstreams:** `od_account_01`, `od_account_02`, etc.
* **Union Remote:** `cloud_union`.
* **Policy Strategy:** `mfs` (Most Free Space) - Writes new files to the account with the most available storage, balancing the load automatically.
* **Mount Strategy:** Systemd services mount the remote to `/mnt/cloud_storage` on the Rocky Linux Host.

---

## 3. Rclone Configuration & Automation

### 3.1 The Union Config
The `rclone.conf` defines the individual remotes and the union policy.

```ini
[cloud_union]
type = union
upstreams = od_account_01: od_account_02:
# Policy: Create files on the remote with the most free space
create_policy = mfs
# Policy: Search on all remotes when reading
search_policy = ff
```

### 3.2 Systemd Mounting
To ensure the drive survives reboots, I created a systemd service file: `/etc/systemd/system/rclone-cloud.service`.

**Critical Flags used:**
* `--allow-other`: Essential for Docker/Samba to see the files (requires `user_allow_other` in `/etc/fuse.conf`).
* `--vfs-cache-mode full`: Simulates a real disk, allowing file seeking and preventing corruption.
* `--umask 000`: Grants full read/write permissions to all users (fixes permission errors).

---

## 4. Service Deployment (Full Docker Stack)

Both the web interface and the SMB server are deployed via a single `docker-compose.yml` stack for easy management and portability.

### 4.1 Docker Compose Configuration
This setup exposes the Rclone mount to the network via standard Docker binds.

```yaml
services:
  # Web Interface (Streaming + File Management)
  # Access via http://<IP>:8085
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser-cloud
    user: "1000:1000"
    ports:
      - "8085:8080"
    volumes:
      - /mnt/cloud_storage:/srv
      - ./filebrowser.db:/database.db
      - ./settings.json:/config/settings.json
    environment:
      - FB_BASE_URL=/
      - FB_DATABASE=/database.db
      - FB_ADDRESS=0.0.0.0
      - FB_ROOT=/srv
    restart: always

  # SMB Network Share for Windows/macOS Clients
  # Access via smb://<IP>/Family_Docs
  samba:
    image: dperson/samba
    container_name: samba-cloud
    environment:
      - USERID=1000
      - GROUPID=1000
      - TZ=Europe/Paris
    ports:
      - "139:139"
      - "445:445"
    volumes:
      - /mnt/cloud_storage:/mnt/data
    # Share Configuration: user;password -s share_name;path;visible;readonly;guest;users
    command: -u "user;password" -s "Family_Docs;/mnt/data;YES;NO;NO;user"
    restart: always
```

---

## 5. Technical Challenges & Fixes

### 5.1 Docker Visibility (FUSE Mounts)
A major hurdle was passing the Rclone FUSE mount from the Host into Docker containers.
* **Problem:** By default, containers cannot access FUSE mounts due to isolation levels.
* **Solution:** Enabling `user_allow_other` in `/etc/fuse.conf` and ensuring the service runs with `--allow-other`. In more complex scenarios (like databases), bind propagation (`:rslave`) was researched to ensure connection stability.

### 5.2 Microsoft API Rate Limits
Creating multiple accounts and simultaneously mounting them triggered Microsoft's anti-bot protection (`403 Forbidden` / `invalid_grant`).
* **Resolution Strategy:**
    1.  **Isolation:** Temporarily removed flagged accounts from the Rclone Union config.
    2.  **Rotation:** Re-authenticated accounts one by one after a 48h cool-down period.
    3.  **Result:** Stabilized the array using "Survivor" accounts while waiting for the others to unlock.

---

## 6. Screenshots

<img width="800" alt="Rclone Mounts Disk Usage" src="https://github.com/user-attachments/assets/0bf0c55a-0d80-4fe4-bba7-4a2a7a5d129a" />
<img width="800" alt="FileBrowser Web Interface" src="https://github.com/user-attachments/assets/5bd5e987-3036-4821-824b-2207fb98eaa4" />
