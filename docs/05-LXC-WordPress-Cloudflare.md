# 05: LXC - WordPress & Cloudflare Tunnel (Via Scripts)

This log documents the deployment of the WordPress instance using Proxmox Community Scripts, specifically configured to work behind a Cloudflare Tunnel without opening external ports.

---

## 1. Objective

Deploy WordPress in a lightweight LXC container, secure it via Proxmox Firewall, and fix the HTTPS routing issues inherent to reverse proxies using specific `wp-config.php` overrides.

---

## 2. Installation (Community Scripts)

Installation was automated using the standard **Proxmox VE Helper-Scripts**.

1.  **WordPress LXC:**
    * Command: `bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/wordpress.sh)"`
    * OS: Debian 13.
    * Web Server: Apache.
    * Network: Static IP assigned.

2.  **Cloudflared LXC:**
    * Command: `bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/cloudflared.sh)"`
    * Authenticated via Cloudflare Zero Trust Token.

---

## 3. Network & Security Architecture

### 3.1 Traffic Flow
`Internet (HTTPS)` -> `Cloudflare Edge` -> `Cloudflared Tunnel` -> `Nginx Proxy Manager (LXC)` -> `WordPress LXC (HTTP Port 80)`

* **Encryption:** SSL is handled by Cloudflare Edge and NPM. The internal connection to the WordPress container is HTTP (Port 80).

### 3.2 Proxmox Firewall Configuration
To isolate the WordPress container, the Proxmox Firewall was enabled on the LXC interface:

* **Input Policy:** DROP
* **Allowed Rule:**
    * **Protocol:** TCP
    * **Dest Port:** 80
    * **Source:** NPMPlus IP
* **Result:** The container is unreachable on any other port, increasing security.

---

## 4. WordPress Configuration (`wp-config.php`)

Since the container listens on Port 80 but the site is accessed via HTTPS, WordPress enters a redirect loop by default.
**Solution:** Edited `/var/www/html/wordpress/wp-config.php` to force HTTPS and define the Site URL explicitly.

**Code added to `wp-config.php`:**

```php
// Force Site URL (prevents database mismatches)
define( 'WP_HOME', 'https://example.com' );
define( 'WP_SITEURL', 'https://example.com' );

// WordPress behind Cloudflare Tunnel (Fixes SSL Redirect Loop)
if (isset($_SERVER['HTTP_CF_VISITOR'])) {
    $visitor = json_decode($_SERVER['HTTP_CF_VISITOR'], true);
    if (isset($visitor['scheme']) && $visitor['scheme'] === 'https') {
        $_SERVER['HTTPS'] = 'on';
    }
}

// Disable Script Concatenation (Fixes Admin Panel display issues)
define( 'CONCATENATE_SCRIPTS', false );
```

## Reverse Proxy Configuration (Nginx Proxy Manager)
* **Proxy Host:** `example.com`
* **Forward IP:** WordPress LXC IP
* **Forward Port:** 80

## 5. Screenshots
<img width="800" alt="Proxmox VE container summary showing low resource usage for the WordPress LXC" src="https://github.com/user-attachments/assets/3f20a153-1bf7-40b7-97af-f11f796a4f5f" />
<img width="800" alt="Proxmox firewall enabled" src="https://github.com/user-attachments/assets/3d0b678a-3171-4f1e-927d-25c4164fe3b0" />
<img width="800" alt="Firewall rules allowing only TCP traffic on port 80" src="https://github.com/user-attachments/assets/922a967e-c348-4765-9808-17243980f083" />
<img width="800" alt="Terminal view of nano editor modifying wp-config.php for Cloudflare SSL compatibility" src="https://github.com/user-attachments/assets/acbe98ba-eea6-43b8-9303-9f649800d410" />
<img width="800" alt="Nginx Proxy Manager configuration screen" src="https://github.com/user-attachments/assets/1ec00489-8c50-4b4b-9873-4a93f99aba86" />
<img width="800" alt="Cloudflare Zero Trust dashboard displaying the active tunnel status as Healthy" src="https://github.com/user-attachments/assets/1cfdb2d6-0c36-4ada-96fc-92d3812448ef" />



