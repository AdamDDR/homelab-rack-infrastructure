# **🛡️ Internal Routing & SSL (Nginx Proxy Manager CT 111\)**

This LXC container serves as the central traffic director. It uses **Nginx Proxy Manager (NPM)** installed directly on Debian (no Docker involved) to provide a web-based GUI for managing reverse proxy hosts, custom headers, and SSL certificates.

* **ID:** 111 | **OS:** Debian 12 | **IP:** 10.0.1.111/23 | **Gateway:** 10.0.0.1

## **1\. Installation (Bare-Metal LXC / No Docker)**

To keep overhead low, NPM is installed natively in an unprivileged LXC using the Proxmox helper scripts (which compiles NPM without requiring Docker).

Run this in your Proxmox Node Shell to provision the container:

bash \-c "$(wget \-qLO \- https://github.com/tteck/Proxmox/raw/main/ct/nginxproxymanager.sh)"

Once installed, access the web GUI at http://10.0.1.111:81.

## **2\. SSL Certificate Strategy (Internal vs. External)**

Managing SSL correctly is critical for Split-Horizon DNS so you don't get browser warnings when accessing services locally.

* **External Traffic (Cloudflare):** External users hit Cloudflare's Edge, which provides a **Universal SSL** certificate. The traffic is encrypted through the cloudflared tunnel, and the tunnel forwards it to NPM via port 80 (HTTP).  
* **Internal Traffic (LAN/Sophos):** Local users bypass Cloudflare entirely via the Sophos DNS entries. Because they hit NPM directly over the LAN using the domain.local URL, NPM needs its own SSL certificate to secure the connection.

### **Linking Cloudflare Certificates to NPM (DNS Challenge)**

To secure local LAN traffic, we configure NPM to pull a valid Let's Encrypt certificate using a Cloudflare API token. This is done via a "DNS Challenge," which means no ports need to be exposed to the internet to verify the certificate.

1. Go to the Cloudflare Dashboard \> My Profile \> API Tokens.  
2. Create a token using the **Edit zone DNS** template for domain.local.  
3. In the NPM GUI (Port 81), go to **SSL Certificates** \> **Add SSL Certificate** \> **Let's Encrypt**.  
4. **Domain Names:** \*.domain.local and domain.local  
5. **Use a DNS Challenge:** Toggle on.  
6. **DNS Provider:** Cloudflare.  
7. Paste your API token in the credentials box and click **Save**.

NPM now holds a valid wildcard certificate to secure all internal traffic\!

## **3\. Configuring Proxy Hosts via GUI**

In the NPM GUI, go to **Hosts \> Proxy Hosts \> Add Proxy Host**.

### **🎬 Jellyfin (Public \+ Local)**

* **Details Tab:**  
  * Domain Names: media.domain.local  
  * Scheme: http | Forward Hostname: 10.0.1.109 | Forward Port: 8096  
  * Enable: Cache Assets, Block Common Exploits, WebSockets Support.  
* **SSL Tab:**  
  * SSL Certificate: Select your \*.domain.local Let's Encrypt cert.  
  * Enable: Force SSL.  
* **Advanced Tab:** (Required to see real client IPs from Cloudflare)  
  real\_ip\_header CF-Connecting-IP;  
  set\_real\_ip\_from 10.0.1.110; \# IP of cloudflared LXC

### **🏡 Home Assistant (Local Internal ONLY)**

Because HA is NOT in the cloudflared config, this rule only applies to traffic originating from inside the LAN (routed by Sophos).

* **Details Tab:**  
  * Domain Names: ha.domain.local  
  * Scheme: http | Forward Hostname: 10.0.0.4 | Forward Port: 8123  
  * Enable: WebSockets Support (CRITICAL for HA).  
* **SSL Tab:**  
  * SSL Certificate: Select your \*.domain.local Let's Encrypt cert.  
  * Enable: Force SSL.
