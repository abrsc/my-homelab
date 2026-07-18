# 18: Microsoft Lab - Hybrid Identity & Entra ID

This log documents the Microsoft hybrid identity lab added to the homelab. The goal is to reproduce a small enterprise-style Microsoft environment combining on-premises Active Directory with Microsoft Entra ID, while keeping sensitive identifiers out of public documentation.

---

## 1. Objective

* **On-Premises Infrastructure:** Deploy Active Directory Domain Services (AD DS) and AD-integrated DNS.
* **Server Administration:** Manage workloads via Windows Admin Center and Hyper-V.
* **Client Management:** Configure domain-joined Windows clients and custom UPN suffixes.
* **Cloud Identity (Entra ID):** Validate custom domains, implement Microsoft Entra Cloud Sync, and enable Password Hash Sync.
* **Hybrid Lifecycle:** Manage users seamlessly from on-prem to the cloud:
  ```text
  Active Directory ➔ Microsoft Entra Cloud Sync ➔ Microsoft Entra ID
  ```

---

## 2. Architecture Overview

```text
Microsoft Hybrid Lab
│
├── Domain Controller
│   ├── Active Directory Domain Services & DNS
│   └── Group Policy processing
│
├── Management Server
│   ├── Windows Admin Center & Hyper-V
│   └── Microsoft Entra Cloud Sync Agent
│
├── Windows Client
│   └── Active Directory domain-joined workstation
│
└── Microsoft Entra ID
    ├── Verified custom domain & Hybrid users
    └── Cloud Sync & Password Hash Sync
```

---

## 3. Network Placement

The Microsoft lab runs in a dedicated isolated lab subnet (`10.10.30.0/24`).

| IP Address | Hostname | Role | Type |
| :--- | :--- | :--- | :--- |
| `.2` | `win-dc01` | Domain Controller / DNS | VM |
| `.3` | `win-mgmt01` | WAC / Hyper-V / Cloud Sync | VM |
| `DHCP` | `win-cli01` | Domain-joined Client | VM |

---

## 4. Core Systems & Responsibilities

The lab uses a **split identity design** (private internal AD namespace vs. public verified UPN suffix for cloud identities). 

### 4.1 Domain Controller (`win-dc01`)
Dedicated exclusively to domain services. Not used as a general-purpose server.
* **Services:** AD DS, AD-integrated DNS.
* **Responsibilities:** Domain authentication, object storage, and Group Policy processing.

### 4.2 Management Server (`win-mgmt01`)
* **Services:** Windows Admin Center, Hyper-V, Cloud Sync Agent.
* **Responsibilities:** Central Windows administration gateway, hosting lab workloads, and acting as the remote administration endpoint.

### 4.3 Windows Client (`win-cli01`)
* **Services:** Domain-joined workstation.
* **Responsibilities:** Validating domain user sign-in, GPO application, and local administrator delegation.

---

## 5. DNS Design & Synchronization

### DNS Resolution
Domain-joined machines are forced to use the Active Directory DNS server instead of public DNS directly, ensuring AD can locate domain controllers.
```text
Domain clients ➔ AD DNS server ➔ AD-integrated DNS zones ➔ Public DNS forwarders
```

### Entra ID Synchronization
Synchronization is controlled by a dedicated **Cloud Sync scope group**.
* **Source of Authority:** Active Directory remains the primary source for account creation, UPN, display name, and passwords.
* **Password Hash Sync:** Enabled, allowing users to authenticate to Microsoft cloud services using their AD-managed password.
* **Clean Design:** OU structure is strictly for administration and GPOs, while *security group membership* dictates the synchronization scope.

---

## 6. Implementation Status

**[✓] Implemented:**
* Active Directory Domain Services & AD-integrated DNS
* Domain controller, Management server & Windows client
* Windows Admin Center & Hyper-V
* Custom UPN suffix & Entra custom domain validation
* Microsoft Entra Cloud Sync & Password Hash Sync (Group-based scope)

**[ ] Planned / Future:**
* Microsoft Intune & Windows Autopilot
* Hybrid Microsoft Entra Join & Conditional Access policies
* Self-Service Password Reset (SSPR) with password/group writeback
* High Availability (Second DC & Cloud Sync agent)

---

## 7. Screenshots

<img width="800" alt="Windows Admin Center dashboard showing successful management connections to the on-premises Domain Controller and Management Server." src="https://github.com/user-attachments/assets/babf2fa5-e3f8-4886-a6ed-f565c2bf6726" />
<img width="800" alt="Microsoft Entra Cloud Sync portal showing a Healthy agent status and Password Hash Sync successfully enabled between the local AD and the cloud." src="https://github.com/user-attachments/assets/ffad2c89-b3b6-4120-96d4-567f203c5dd4" />
<img width="800" alt="List of users in Microsoft Entra ID with the 'On-premises sync enabled' attribute set to Yes, confirming successful hybrid identity provisioning." src="https://github.com/user-attachments/assets/19438b9d-e8a6-40c7-88ee-afe19fb09308" />

