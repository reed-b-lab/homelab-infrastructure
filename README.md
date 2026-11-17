# homelab-infrastructure
Self-hosted Debian home lab featuring Docker Compose stacks (Pi-hole, Gluetun VPN, qBittorrent, Jellyfin) and KVM/libvirt multi-OS virtualization.
# Home Lab Infrastructure & Virtualization Stack

Documentation, Docker Compose stacks, and KVM/libvirt configurations for my self-hosted Debian home server and multi-OS security lab.

## Architecture Overview

* **Host OS:** Debian Linux running Docker Engine and KVM/QEMU (`libvirt`).
* **Network & DNS (`pihole/`):** Containerized Pi-hole providing network-wide DNS filtering and local DNS resolution.
* **VPN-Isolated Media Stack (`media-stack/`):**
  * **Gluetun:** WireGuard VPN client container acting as an isolated network gateway.
  * **qBittorrent:** Routed strictly through Gluetun (`network_mode: "container:gluetun"`) with health-check startup ordering so torrent traffic never leaks or loses `tun0` on host reboot.
  * **Jellyfin:** Hardware-accelerated media server (`/dev/dri` passthrough) reading from shared storage.
* **Virtualization & Security Lab (`virtualization/`):**
  * Headless KVM hypervisor managed remotely via `virt-manager` and `virt-viewer` over SSH (`qemu+ssh://`).
  * Multi-OS lab topology (**Windows 11**, **Kali Linux**, and **Ubuntu**) across NAT (`virbr0`) and isolated host-only virtual networks for sysadmin break/fix drills and offensive/defensive security testing.

## Quick Start (Docker Stacks)

1. Copy the example environment file in each stack directory and fill in your values:
   ```bash
   cp media-stack/.env.example media-stack/.env
   cp pihole/.env.example pihole/.env
   ```
2. Launch the services:
   ```bash
   docker compose -f pihole/docker-compose.yml up -d
   docker compose -f media-stack/docker-compose.yml up -d
   ```

## Security Practices

* All sensitive credentials, WireGuard private keys, and local subnet ranges are abstracted into `.env` files (excluded via `.gitignore`).
* `qbittorrent` is bound to Gluetun's `tun0` interface and waits for `service_healthy` before starting.
* SSH access to the hypervisor and VMs is restricted to the local management subnet via `ufw`.
