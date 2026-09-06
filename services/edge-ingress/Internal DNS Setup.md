# **🧭 Internal DNS Orchestration (Sophos VM 201\)**

Sophos acts as the primary network gateway (10.0.0.1) and local DNS resolver.

## **Split-Horizon DNS Configuration**

Split-Horizon forces local devices to route traffic directly to the internal Nginx Proxy (10.0.1.111), completely bypassing the internet and Cloudflare Tunnels.

**Benefits:**

1. **Jellyfin:** Streams media locally at full switch speeds (1 Gbps) without consuming WAN bandwidth.  
2. **Home Assistant:** Provides secure, LAN-only access using standard domain names (ha.domain.local).

## **Setup Steps (Sophos Web GUI)**

1. Log into Sophos at https://10.0.0.1.  
2. Navigate to **Network \> DNS**.  
3. Under **DNS Host Entries**, click **Add** to create the following records:

| Host Name | IP Address | Routing Result |
| :---- | :---- | :---- |
| media.domain.local | 10.0.1.111 | Re-routes Jellyfin traffic to internal proxy |
| ha.domain.local | 10.0.1.111 | Routes internal Home Assistant traffic to proxy |

*Note: All domain entries must point to 10.0.1.111 (Nginx), not the specific container IP. Nginx is responsible for looking at the domain name and forwarding it to 10.0.0.4 or 10.0.1.109.*
