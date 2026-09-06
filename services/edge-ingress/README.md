# **🌐 Edge Routing, DNS Orchestration & Reverse Proxy Architecture**

> **Sanitization Notice:** All public domains are masked as domain.local (and \*.domain.local), and IP addresses are scrubbed to the internal 10.0.0.0/23 subnet for privacy.

This repository documents the bare-metal LXC/VM routing infrastructure on the batcaveserver Proxmox node.

## **1\. Physical & Virtual Layout**

* **Subnet & Switch Topology:** Flat Layer 2 subnet (10.0.0.0/23) due to hardware switch limitations.  
* **Proxmox Node Layout:** batcaveserver organizes resources by their last octet matching the Proxmox ID.  
* **No Docker:** cloudflared and Nginx Proxy Manager (NPM) run in native, unprivileged Proxmox LXC containers.

| Service | Proxmox ID / Type | IP Address | Subnet | Gateway | Exposure |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Sophos-OS** | VM 201 | 10.0.0.1 | /23 | N/A | Gateway / DNS |
| **HAOS** | VM 204 | 10.0.0.4 | /23 | 10.0.0.1 | **Internal Only** |
| **Jellyfin** | CT 109 | 10.0.1.109 | /23 | 10.0.0.1 | Public (via Tunnel) |
| **cloudflared** | CT 110 | 10.0.1.110 | /23 | 10.0.0.1 | Public Ingress |
| **NPM (Nginx)** | CT 111 | 10.0.1.111 | /23 | 10.0.0.1 | Internal Proxy |

## **2\. Traffic Flow & SSL Routing**

### **Flow A: Public Ingress (External Users \-\> Jellyfin)**

External clients hit Cloudflare's Edge, where **Cloudflare's Universal SSL** handles the public encryption. Traffic travels through the encrypted tunnel to the cloudflared LXC (10.0.1.110), which proxies it via standard HTTP to NPM (10.0.1.111). NPM routes it to Jellyfin.

Client ➔ Cloudflare Edge (SSL) ➔ cloudflared (CT 110\) ➔ NPM (CT 111\) ➔ Jellyfin (CT 109\)

### **Flow B: Internal Split-Horizon DNS (Local Users \-\> HA & Jellyfin)**

Internal devices querying \*.domain.local are intercepted by the Sophos Firewall DNS and pointed directly to the NPM container, bypassing the internet. To prevent browser security warnings, NPM uses a **Let's Encrypt wildcard certificate** (generated via Cloudflare DNS challenge) to handle the local SSL handshake.

Local Client ➔ Sophos DNS (10.0.0.1) ➔ NPM (CT 111 \- Let's Encrypt SSL) ➔ Target App

## **📚 Setup Guides**

* **Public Routing:** [Namecheap, Cloudflare & Tunnel Setup](services/edge-ingress/Public.md)  
* **Internal Routing & SSL:** [Nginx Proxy Manager Setup](services/edge-ingress/README.md)  
* **Internal DNS:** [Sophos Split-Horizon Setup](services/edge-ingress/Internal-DNS-Setup.md)
