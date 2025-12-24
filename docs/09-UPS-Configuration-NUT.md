# 09: Power Protection – UPS & NUT Configuration

This log documents the installation of **Network UPS Tools (NUT)** to manage a **Salicru SPS SOHO+ 850VA** UPS.
The goal is to ensure **automated, graceful shutdowns** for both the Hypervisor (Master) and the Backup Server (Slave) during power outages.

---

## 1. Hardware

- **Device:** Salicru SPS SOHO+ 850VA
- **Type:** Line-interactive (with AVR)
- **Status:** New Unit
- **Connection:** USB to Proxmox Host (PVE - 192.168.1.100)
- **Driver:** `usbhid-ups`

---

## 2. Master Configuration (PVE Node)

The Proxmox VE node acts as the **NUT Master**. It connects directly to the UPS via USB and broadcasts the status to the network.

### 2.1 Driver Configuration (`/etc/nut/ups.conf`)
Defined specific safety overrides to force shutdown at 25% battery.

```ini
[ups]
    driver = usbhid-ups
    port = auto
    desc = "Salicru SPS 850VA"
    vendorid = 06da
    productid = ffff

    # Critical Safety Overrides
    ignorelb
    override.battery.charge.low = 25

    # Shutdown Timers (seconds)
    offdelay = 20
    ondelay = 30
```

### 2.2 Network Listening (`/etc/nut/upsd.conf`)
**Crucial:** Configured the daemon to listen on the LAN interface so the Backup Server (Slave) can connect. Without this, remote connections are refused.

```ini
LISTEN 127.0.0.1 3493
LISTEN 192.168.1.100 3493
```

### 2.3 User Access (`/etc/nut/upsd.users`)
Created an admin user for monitoring.

```ini
[admin]
    password = secret
    actions = SET
    instcmds = ALL
```
*(Note: Replace `secret` with a strong password in production).*

### 2.4 Mode (`/etc/nut/nut.conf`)
```ini
MODE=netserver
```

### 2.5 Local Monitor (`/etc/nut/upsmon.conf`)
The master also monitors itself.
```ini
MONITOR ups@localhost 1 admin secret master
```

---

## 3. Slave Configuration (PBS Node)

The Backup Server (192.168.1.99) is configured as a **Network Client**. It monitors the UPS status remotely via the LAN.

### 3.1 Installation
```bash
apt update
apt install nut-client -y
```

### 3.2 Connection (`/etc/nut/upsmon.conf`)
Configured to listen to the Master node securely as a slave.
*Note: The `slave` flag ensures this node shuts down but does not command the UPS to cut power.*

```ini
MONITOR ups@192.168.1.100 1 admin secret slave
```

### 3.3 Mode (`/etc/nut/nut.conf`)
```ini
MODE=netclient
```

---

## 4. Testing & Verification

### 4.1 Master Check (PVE)
```bash
upsc ups
```

### 4.2 Slave Check (PBS)
Tested connectivity from the PBS node to the PVE node:
```bash
upsc ups@192.168.1.100
```
**Result:** Successful retrieval of battery status (Charge, Voltage, etc.).

### 4.3 Behavior Logic
- **Battery < 25%:** NUT triggers the shutdown command.
- **Sequence:**
    1.  **PBS (Slave)** detects the low battery status from PVE and shuts down immediately.
    2.  **PVE (Master)** waits for the slave to disconnect (or a timeout), stops its own VMs/CTs, and powers off.
    3.  **UPS** cuts power (if configured) or waits for AC restoration.

---

## 5. Screenshots

<img width="800" alt="upsc ups result" src="https://github.com/user-attachments/assets/6b09f55b-912e-48e7-96df-e166dea1ef0b" />
