# 04: LXC - Nginx Proxy Manager Plus (NPMPlus)

This log documents the deployment of Nginx Proxy Manager Plus (NPMPlus) as an internal-only reverse proxy.

---

## 1. Objective

To deploy an internal reverse proxy for two main reasons:

1. Simplify access to homelab services using clean, memorable hostnames (e.g., `proxmox.lan`) instead of IPs and ports.
2. Encrypt all internal web traffic (especially for passwords) using a custom self-signed SSL certificate.

---

## 2. Installation (Proxmox Shell)

Used the community-script to create the LXC and install NPM in a single command from the Proxmox host shell:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/npmplus.sh)"
```

- After the installation I assigned a static IP (`192.168.1.51`).
- The script generated a default admin user. Credentials were found in the LXC console (at `/opt/.npm_pwd`).
- The first login to the web interface immediately required changing these default credentials.

---

## 3. DNS Configuration (AdGuard)

To route all `.lan` traffic to the NPM container, a single wildcard DNS rewrite rule was added in AdGuard (Filters → DNS Rewrites):

- **Domain:** `*.lan`
- **Response:** `192.168.1.51` (the IP of the NPM container)

This setup means AdGuard only needs one rule, and all future hosts can be managed directly in NPM.

---

## 4. Wildcard SSL Certificate Creation

To encrypt local traffic, a self-signed certificate with the correct **Subject Alternative Name (SAN)** was created.  
Modern browsers require SANs, not just a Common Name (CN), to validate a certificate.

### 4.1 Create `openssl.cnf`

```ini
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
CN = *.lan

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = *.lan
DNS.2 = lan
```

### 4.2 Run OpenSSL command (Git Bash)

Generates key + cert valid for 10 years (3650 days):

```bash
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
  -keyout lan.key \
  -out lan.crt \
  -config openssl.cnf \
  -extensions v3_req
```

### 4.3 Upload to NPM

Upload `lan.key` and `lan.crt` to:

**SSL Certificates → Add Custom Certificate**

---

## 5. Proxy Host Configuration (NPM UI)

Two hosts were created to manage the core services:

### A. proxmox.lan

- **Domain Name:** `proxmox-ve.lan`
- **Scheme:** `https`
- **Forward Hostname:** `192.168.1.100`
- **Forward Port:** `8006`
- **SSL:** Assigned the custom `*.lan` certificate and enabled **Force SSL**

### B. adguard.lan

- **Domain Name:** `adguard-home.lan`
- **Scheme:** `http`
- **Forward Hostname:** `192.168.1.50`
- **Forward Port:** `80`
- **SSL:** Assigned the custom `*.lan` certificate and enabled **Force SSL**

---

## 6. Screenshots

<img width="800" alt="dns-rewrite" src="https://github.com/user-attachments/assets/75834057-c352-4f93-bded-b53b79360683" />
<img width="800" alt="proxy-hosts" src="https://github.com/user-attachments/assets/b67f30f8-1b50-45d7-90db-e89494eaefa5" />
<img width="800" alt="imagen" src="https://github.com/user-attachments/assets/97d86c27-10f2-4d62-81c5-5a3e41e6383d" />

