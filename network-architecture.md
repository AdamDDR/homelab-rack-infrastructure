# 🌐 Network Architecture & Segmentation Matrix

## 1. Addressing Scheme (RFC 1918)

| VLAN ID | Subnet Name | CIDR Prefix | Gateway | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | Management | `10.10.10.0/24` | `10.10.10.1` | Proxmox VE host, managed switch web interfaces, PDU. |
| **VLAN 20** | Trusted LAN | `10.10.20.0/24` | `10.10.20.1` | Daily workstations, administrative laptops, trusted Wi-Fi. |
| **VLAN 30** | IoT / Smart Home | `10.10.30.0/24` | `10.10.30.1` | ESP32s, ESP8266s, Sonoff switches, Zigbee gateways. Isolated from WAN. |
| **VLAN 40** | DMZ / Container Services | `10.10.40.0/24` | `10.10.40.1` | Docker containers, Nginx Proxy Manager, media servers. |

---

## 2. Firewall Inter-Zone Rules (Sophos Firewall)
[VLAN 20: Trusted LAN] ---------> ALLOW ALL ---------> [All Internal VLANs & Internet]
[VLAN 30: IoT Network] ---------> DROP ---------------> [VLAN 10, VLAN 20, VLAN 40]
[VLAN 30: IoT Network] ---------> RESTRICTED ---------> [NTP / Local Home Assistant Core Only]
[VLAN 40: Services] -----------> ESTABLISHED/RELATED -> [VLAN 20]
[VLAN 40: Services] -----------> DROP ---------------> [VLAN 10: Management]
* **Default Deny**: All inter-VLAN traffic is blocked by default unless explicitly allowed by stateful firewall policies.
* **IoT Isolation**: IoT devices cannot initiate connections to any other VLAN or the public internet, preventing rogue firmware telemetry or lateral movement.
