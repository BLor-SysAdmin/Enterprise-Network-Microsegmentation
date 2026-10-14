# Enterprise Network Microsegmentation & Perimeter Access Control Policies

## Executive Architecture Overview
This network security infrastructure portfolio showcases the deployment of a micro-segmented virtual network topology engineered to enforce a strict Zero Trust model. Flat enterprise networks present an extreme risk vector, allowing an attacker who compromises an edge client to freely move laterally into private server zones. This project mitigates that risk by implementing layer-2 virtual network fabric boundaries (802.1Q VLANs) managed by a multi-interface core firewall gateway, explicitly isolating production assets from untrusted zones.

## System Topology Blueprint
*   **Core Security Appliance:** OPNsense Firewall Engine (FreeBSD Kernel Baseline)
*   **Virtual Switch Switching Fabric:** Proxmox VE VirtIO Bridges (`vmbr0` WAN, `vmbr1` Production LAN, `vmbr2` Isolated Labs)
*   **Network Segments Implemented:**
    *   **Management / LAN Zone:** `10.0.1.0/24` (Houses `DC-01`, Grafana, and Admin Workstations)
    *   **Isolated Client Zone (VLAN 50):** `10.0.50.0/24` (Hosts untrusted endpoints and guest devices)
    *   **VPN Road Warrior Pool:** `10.50.0.0/24` (Secure cryptographic remote transit network)

## Technical Features Implemented
1.  **Layer-2 802.1Q VLAN Segmentation:** Provisioned a virtualized switch tag engine on the core firewall interface loop to instantiate a distinct network interface domain (**VLAN 50**). This encapsulates guest and untrusted workstation frames, preventing untagged inter-subnet visibility at the switch level.
2.  **Top-Down State-Enforced Access Control Lists (ACLs):** Configured strict packet inspection rules inside the OPNsense routing matrix. Enforced defensive implicit-deny logic that permits VLAN 50 traffic to route outwards to the public WAN for external internet access, while explicitly logging and dropping any packets targeting the secure corporate `10.0.1.x` infrastructure assets.
3.  **RFC 1918 Private Block Suppression:** Activated strict network edge filter rules on the primary WAN interface block, forcing the automatic deletion of incoming bogon networks and unroutable private IP addresses to completely eliminate external perimeter spoofing vectors.

## Core Routing Access Policy Logic Matrix

| Firewall Interface Rule Group | Action | Protocol | Source Address Pool | Destination Address Target | Intended Security Outcome |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **WAN Interface Rule** | Block | Any | Bogon / Private Networks | Any | Drops unroutable traffic arriving from the public internet. |
| **LAN Management Interface**| Pass | Any | `10.0.1.0/24` (Admin Zone) | Any | Permits internal systems admins to manage firewalls and infrastructure servers. |
| **VLAN 50 (Isolated Users)** | **Block**| Any | `10.0.50.0/24` (Guest Zone) | `10.0.1.0/24` (Corporate Core) | **Seals off critical directory and monitoring servers from untrusted client computers.** |
| **VLAN 50 (Isolated Users)** | Pass | Any | `10.0.50.0/24` (Guest Zone) | WAN net (Public Internet) | Grants user internet navigation access while keeping them sandboxed from core lab assets. |