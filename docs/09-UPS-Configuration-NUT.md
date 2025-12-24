# 09: Power Protection – UPS & NUT Configuration

This log documents the installation of **Network UPS Tools (NUT)** to manage a **Salicru SPS SOHO+ 850VA** UPS.
The goal is to ensure **automated, graceful shutdowns** during power outages to prevent data corruption.

---

## 1. Hardware

- **Device:** Salicru SPS SOHO+ 850VA
- **Type:** Line-interactive (with AVR - Automatic Voltage Regulation)
- **Status:** New Unit
- **Connection:** USB to Proxmox Host
- **Driver:** `usbhid-ups`

---

## 2. Installation

Installed the standard NUT package from the Debian repositories:

```bash
apt update
apt install nut -y
```

---

## 3. Device Identification

To configure the driver correctly, I needed to find the specific USB identifiers of the UPS.
I used the `lsusb` command to list connected devices:

```bash
lsusb
```

**Output:**
```text
Bus 001 Device 004: ID 06da:ffff Phoenixtec Power Co., Ltd
```
*(Note: Salicru uses Phoenixtec internals, hence the name).*

This confirmed the IDs needed for configuration:
* **Vendor ID:** `06da`
* **Product ID:** `ffff`

---

## 4. Driver Configuration (`/etc/nut/ups.conf`)

I configured the driver using the IDs found above and added specific safety overrides.
To ensure the system has ample time to stop all VMs cleanly before the battery runs out, I force the shutdown threshold to **25%**.

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

### Notes

- **`ignorelb`**
  Ignores the UPS hardware *Low Battery* signal to rely on the software percentage value instead.

- **`override.battery.charge.low = 25`**
  Forces NUT to initiate shutdown once battery drops below 25%. This creates a safe buffer for the OS to commit data to disks.

---

## 5. User Configuration (`/etc/nut/upsd.users`)

Defined an administrative user to allow the monitoring daemon (`upsmon`) to communicate with the UPS server locally.

```ini
[admin]
    password = secret
    actions = SET
    instcmds = ALL
```
*(Note: Replace `secret` with a strong password in production).*

---

## 6. Monitor Configuration

### 6.1 Mode (`/etc/nut/nut.conf`)

```ini
MODE=netserver
```

### 6.2 Monitor (`/etc/nut/upsmon.conf`)

Configured the local monitoring client to authenticate with the user created in Step 5.

```ini
MONITOR ups@localhost 1 admin secret master
```

---

## 7. Testing & Verification

### 7.1 Status Check

```bash
upsc ups
```

### 7.2 Behavior Logic

- **Voltage Dip/Spike**
  UPS uses AVR (Automatic Voltage Regulation) to stabilize output without switching to battery.

- **Total Power Loss**
  UPS switches to battery → NUT detects `OB` (*On Battery*).

- **Battery < 25%**
  NUT triggers the shutdown command because of the override settings.

- **Shutdown Sequence**
  Proxmox gracefully stops all VMs/CTs, then powers off the host.

This ensures data integrity and prevents filesystem corruption.

## 8. Screenshots

<img width="800" alt="upsc ups result" src="https://github.com/user-attachments/assets/6b09f55b-912e-48e7-96df-e166dea1ef0b" />
