# 16: AI Stack - GPU Passthrough & Local LLM Services

This log documents the high-performance integration of an **NVIDIA RTX 3060 12GB** into the Proxmox cluster using LXC Passthrough to power a private AI ecosystem.

---

## 1. Objective

Leverage the 12GB of VRAM for Large Language Models (LLMs) while maintaining the efficiency of Linux Containers (LXC).
* **Hardware:** NVIDIA GeForce RTX 3060 12GB.
* **Architecture:** Shared Kernel Passthrough (zero-latency compute).

---

## 2. GPU Passthrough Setup (Proxmox Host)

The host (Proxmox VE 9.1) was configured to share the GPU with the container ID `117`.

1.  **Host Drivers:** Installed NVIDIA proprietary drivers on the PVE node.
2.  **Persistence:** Enabled `nvidia-smi -pm 1` to keep the GPU initialized.
3.  **LXC Hooks:** Modified `/etc/pve/lxc/117.conf` to map `/dev/nvidia*` devices and allow cgroup2 access.

---

## 3. Local AI Engine (Ollama)

Installed inside the GPU-enabled LXC (`192.168.1.104`).

* **Driver Subtlety:** Installed the NVIDIA driver with the `--no-kernel-module` flag to use the host's existing kernel module.
* **Networking:** Modified the systemd service with `Environment="OLLAMA_HOST=0.0.0.0"` to allow remote API calls.
* **Verification:** `nvidia-smi` inside the LXC correctly displays the GPU and its VRAM.

---

## 4. Web Interface (Open WebUI)

Deployed via **Docker Compose** on the Docker VM (`192.168.1.60`) to provide a ChatGPT-like experience.

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    environment:
      - 'OLLAMA_BASE_URL=[http://192.168.1.104:11434](http://192.168.1.104:11434)' # Points to the GPU LXC
    volumes:
      - open-webui-storage:/app/backend/data
```

---

## 5. Summary of Achievements

* **Privacy:** All AI queries stay within the local network.
* **Cost:** $0 subscription fees; utilizing existing workstation hardware.
* **Security:** Access to the UI is secured via the internal network and optional Cloudflare Zero Trust tunnel.

---

## 6. Screenshots
<img width="800" alt="nvidia-smi output" src="https://github.com/user-attachments/assets/da92430f-e594-44fa-9e42-ad94b1f2da51" />
<img width="800" alt="OpenWebUI" src="https://github.com/user-attachments/assets/64d0b562-6773-432e-9c1d-5703de1d3c83" />
<img width="800" alt="LXC Config File" src="https://github.com/user-attachments/assets/10ab00df-a05c-4c9c-8d90-1725a4dd1cf5" />


