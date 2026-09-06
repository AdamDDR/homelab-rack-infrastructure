# **☁️ Public Ingress & Cloudflare Orchestration**

This document covers the public-facing stack, ensuring secure remote access to Jellyfin (media.domain.local) while completely isolating internal tools (like Home Assistant) from the web.

## **1\. Domain Registration & Delegation (Namecheap)**

1. Purchase your domain (domain.local) via Namecheap.  
2. In Namecheap: **Domain List \> Manage \> Nameservers**.  
3. Select **Custom DNS** and enter your assigned Cloudflare nameservers (e.g., dina.ns.cloudflare.com and phil.ns.cloudflare.com).

## **2\. SSL & Edge Setup (Cloudflare Dashboard)**

When you delegate to Cloudflare, Cloudflare automatically provisions a **Universal SSL Certificate** for the edge.

* External users connecting to https://media.domain.local will perform an SSL handshake with Cloudflare's servers.  
* Go to **SSL/TLS \> Overview** and ensure your encryption mode is set to **Full (Strict)**.

### **Caching Rules (CRITICAL for Jellyfin)**

Cloudflare's Terms of Service strictly prohibit disproportionate non-HTML content (like video streaming) via their caching network. You must bypass the cache to prevent domain suspension.

1. Cloudflare Dashboard \> **Caching \> Cache Rules**.  
2. Create rule: **"Bypass Jellyfin Media"**.  
3. **If...** Hostname equals media.domain.local \-\> **Then...** Cache status: **Bypass cache**.

## **3\. cloudflared LXC Configuration (CT 110\)**

This LXC acts as the outbound secure tunnel. It connects to Cloudflare and forwards incoming traffic to the internal Nginx Proxy Manager (NPM).

* **ID:** 110 | **OS:** Debian 12 | **IP:** 10.0.1.110/23 | **Gateway:** 10.0.0.1

### **Installation**

apt update && apt upgrade \-y  
curl \-L 'https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb' \-o cloudflared.deb  
dpkg \-i cloudflared.deb  
cloudflared tunnel login  
cloudflared tunnel create batcave-tunnel

### **Routing Configuration (/etc/cloudflared/config.yml)**

Only public services are defined here. Home Assistant is strictly excluded.

tunnel: \<YOUR\_TUNNEL\_UUID\>  
credentials-file: /root/.cloudflared/\<YOUR\_TUNNEL\_UUID\>.json

ingress:  
  \# Route Jellyfin traffic to the NPM LXC  
  \- hostname: "media.domain.local"  
    service: http://10.0.1.111:80  
    originRequest:  
      httpHostHeader: "media.domain.local"  
    
  \# Catch-all  
  \- service: http\_status:404

### **Start Service**

cloudflared service install  
systemctl start cloudflared && systemctl enable cloudflared  
