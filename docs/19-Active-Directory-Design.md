# 19: Active Directory Design & Administration

This log documents the Active Directory structure, Group Policy layout, and delegated administration model used in the Microsoft hybrid identity lab. 

*Note: Sensitive details (real domain names, real usernames, exact distinguished names) are intentionally omitted from public documentation.*

---

## 1. Objective

Build a clean Active Directory structure that is simple enough for a homelab but strictly follows enterprise-style principles.

* **Separation:** Clearly isolate standard users, admins, groups, servers, and workstations.
* **Targeting:** Use OUs strictly for organization and Group Policy targeting.
* **RBAC:** Use groups to define roles, permissions, and synchronization scopes.
* **Security First:** Avoid using high-privilege accounts for routine administration.

---

## 2. OU Structure

OUs are not used to represent every application state. They serve to structure the directory and apply policies effectively.

```text
Domain root
│
├── Domain Controllers
│   └── Domain Controller
│
└── LAB
    ├── Admins           (Named administrative accounts)
    ├── Groups           (Global roles & Domain Local permission groups)
    ├── Servers          (Management servers)
    ├── Service Accounts (Reserved for future technical accounts)
    ├── Users            (Standard users)
    └── Workstations     (Domain-joined clients)
```

---

## 3. Administrative Model

* **Built-in Domain Administrator:** Strictly reserved for high-privilege operations (initial configuration, DC operations, emergency access). Not used for daily tasks.
* **Named Administrative Accounts:** Used for routine delegated administration (server management, Hyper-V, workstations, WAC access).
* **Standard Users:** Reside in the `Users` OU. Their synchronization to Entra ID is completely decoupled from their OU location and relies entirely on their membership in the Cloud Sync scope group.

---

## 4. Group Model (AGDLP-Inspired)

The lab uses an **AGDLP** approach (Account ➔ Global Group ➔ Domain Local Group ➔ Permission) to make permissions easy to audit and extend.

* **Global Groups (Roles):** Identify *who* has a specific role (e.g., `Server administrators`, `Hyper-V administrators`, `Cloud Sync scoped users`).
* **Domain Local Groups (Permissions):** Represent permissions on specific resources acting as the final permission layer (e.g., `Local administrators on the management server`).

### Permission Chains in Action:

**Server Administration:**
```text
Named admin account ➔ Server admins (Global) ➔ Mgmt server local admins (Domain Local) ➔ Local Admin rights
```

**Hyper-V Administration:**
```text
Named admin account ➔ Hyper-V admins (Global) ➔ Hyper-V admins (Domain Local) ➔ Hyper-V rights
```

**Microsoft Entra Cloud Sync Scope:**
*(Note: This is a synchronization scope group, not a server permission group)*
```text
Standard user ➔ Cloud Sync scope group (Global) ➔ Microsoft Entra Cloud Sync
```

---

## 5. Group Policy Layout (GPOs)

GPOs are explicitly separated between workstations and servers to ensure clean policy application.

**Workstations OU Links:**
```text
Workstations OU
├── Workstation baseline policy
└── Workstation local administrators policy
```

**Servers OU Links:**
```text
Servers OU
└── Server baseline policy
```

---

## 6. Validation & Future Improvements

**Validated:**
* Workstation and Server GPO baselines apply correctly to their respective targets.
* Local administrator delegation works flawlessly through nested AD group membership.

**Future Improvements (As needed):**
* *Policies:* User baseline policy, Domain security baseline, Server audit baseline.
* *Structure:* Staging OU, Quarantine OU (only to be added when a true boundary need arises).

---

## 7. Screenshots

<img width="800" alt="Active Directory Users and Computers console showing a clean, enterprise-standard Organizational Unit (OU) structure separating Admins, Groups, Servers, Users, and Workstations." src="https://github.com/user-attachments/assets/7fed9b02-fa8c-4026-833e-ebecdc7fd819" />
<img width="800" alt="Group Policy Management console demonstrating correct GPO targeting, with baseline policies linked explicitly to the Workstations OU rather than the domain root." src="https://github.com/user-attachments/assets/d9087d1d-7504-419e-a904-d5c7d82ca71e" />
<img width="800" alt="Command prompt on a domain-joined Windows 11 client running gpresult /r, confirming that the workstation and local administrator baseline GPOs are successfully applied." src="https://github.com/user-attachments/assets/8be5034a-8a8d-48bf-a167-2fbe4379dabd" />


