# 🏢 Enterprise Homelab & Rack Infrastructure

[![Infrastructure](https://img.shields.io/badge/Infrastructure-Proxmox%20VE-E57000?style=flat-square&logo=proxmox&logoColor=white)](https://www.proxmox.com)
[![Firewall](https://img.shields.io/badge/Firewall-Sophos-006699?style=flat-square&logo=sophos&logoColor=white)](https://www.sophos.com)
[![Edge Routing](https://img.shields.io/badge/Edge-Cloudflare%20Tunnels-F38020?style=flat-square&logo=cloudflare&logoColor=white)](https://cloudflare.com)
[![VPN Mesh](https://img.shields.io/badge/Overlay-Tailscale%20WireGuard-24292E?style=flat-square&logo=tailscale&logoColor=white)](https://tailscale.com)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

> A modular, production-grade private cloud and network rack. Engineered with isolated VLAN segmentation, zero-trust edge ingress, local-first IoT microcontrollers, and automated container orchestration.

---

## 📐 Network Topology & Traffic Flow
## 🗂️ Repository & Sub-Project Directory

This repository serves as the central hub. Detailed configuration manifests and runbooks for individual services live in dedicated subdirectories:

| Subsystem / Service | Description | Directory Link |
| :--- | :--- | :--- |
| **Network Architecture** | Subnet matrix, VLAN definitions, and inter-zone firewall rules. | [`/docs/network-architecture.md`](docs/network-architecture.md) |
| **Physical Rack Specs** | Hardware list, patch panel terminations, and VDSL2 copper diagnostics. | [`/docs/rack-specifications.md`](docs/rack-specifications.md) |
| **2.5G ONT Bypass** | Future roadmap for GPON SFP+ MAC/VLAN spoofing to eliminate double-NAT. | [`/docs/future-roadmap/2.5g-ont-bypass.md`](docs/future-roadmap/2.5g-ont-bypass.md) |
| **Sophos Firewall** | Firewall access policies, NAT rules, and isolated subnets. | [`/services/sophos-firewall/`](services/sophos-firewall/) |
| **Proxmox Virtualization** | KVM storage pools, unprivileged LXC templates, and backup schedules. | [`/services/proxmox-hypervisor/`](services/proxmox-hypervisor/) |
| **Edge Ingress & Reverse Proxy** | Cloudflare Tunnel daemon and Nginx Proxy Manager configurations. | [`/services/edge-ingress/`](services/edge-ingress/) |
| **Tailscale Mesh Overlay** | Zero-trust admin access policies, ACL tags, and subnet routers. | [`/services/tailscale-mesh/`](services/tailscale-mesh/) |
| **Media & Automation Stack** | Docker Compose manifests for Jellyfin, Sonarr, Radarr, and Prowlarr. | [`/services/servarr-media-stack/`](services/servarr-media-stack/) |
| **Local IoT & Hardware** | Home Assistant, ESPHome firmware configs, and Zigbee coordinator setup. | [`/services/iot-smart-home/`](services/iot-smart-home/) |

---

## 🔒 Security, Privacy & Sanitization Policy

This repository is maintained for documentation, architectural sharing, and portfolio review:
* **No Real WAN IPs or Secrets**: All IP addresses adhere strictly to RFC 1918 private ranges (`10.10.0.0/16`).
* **Environment Files**: All credentials, tokens, and keys are scrubbed. Deployments use `.env.example` templates.
* **Public Ingress Model**: No public ports (80/443) are forwarded on the edge router. All inbound public traffic is strictly brokered through Cloudflare Tunnels with WAF rate limiting.
