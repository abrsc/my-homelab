# 07: VM - Docker & Uptime Kuma (Rocky Linux)

This log documents the deployment of a Docker host using the previously created Rocky Linux template, and the setup of Uptime Kuma for monitoring.

---

## 1. Objective

Deploy a production-ready Docker environment on an Enterprise Linux OS (Rocky 10) to host the monitoring stack.
* **Source:** Cloned from "Rocky 10 Template" (see Log 02).
* **Access:** Secured via SSH Key (Cloud-Init).
* **Service:** Uptime Kuma (via Docker Compose).

---

## 2. VM Configuration

* **OS:** Rocky Linux 10.
* **CPU:** 2 vCPU (Type: Host, for better performance).
* **RAM:** 2GB.
* **Storage:** 15GB.
* **Auth:** Public SSH Key injected via Proxmox Cloud-Init settings.

---

## 3. Docker Installation (Rocky/RHEL Method)

Since this VM uses Rocky Linux, I used `dnf` and the official Docker repository for RHEL/CentOS.

1.  **Add Docker Repository:**
    ```bash
    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    ```
2.  **Install Docker Engine:**
    ```bash
    dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    ```
3.  **Start & Enable Service:**
    ```bash
    systemctl start docker
    systemctl enable docker
    ```

---

## 4. Uptime Kuma Deployment (Docker Compose)

I organized the stack in a dedicated directory structure for easier maintenance.

1.  **Directory Setup:**
    ```bash
    mkdir -p ~/stacks/uptime-kuma
    cd ~/stacks/uptime-kuma
    nano compose.yml
    ```

2.  **Compose File Content (`compose.yml`):**
    ```yaml
    services:
      uptime-kuma:
        image: louislam/uptime-kuma:2
        container_name: uptime-kuma
        restart: always
        ports:
          - "3001:3001"
        volumes:
          - ./data:/app/data
    ```

3.  **Deployment:**
    ```bash
    docker compose up -d
    ```

---

## 5. Reverse Proxy Configuration (NPM)

To secure access via HTTPS:

* **Domain:** `uptime-kuma.lan`
* **Target:** `http://192.168.1.60:3001`
* **SSL:** Enabled via the custom wildcard certificate (`*.lan`).

---

## 6. Screenshots
<img width="800" alt="imagen" src="https://github.com/user-attachments/assets/713a2591-5ea5-4d47-8a28-696c6cb9ac4b" />
