# Secure Network Architecture Design (Cisco Packet Tracer)

## Project Overview
This project demonstrates a secure network architecture design for a fictional logistics branch ("DXB-Logistics"). The goal was to implement strict network segmentation using VLANs and Access Control Lists (ACLs) to isolate public Guest traffic from sensitive HR/Finance data.

**Tools Used:** Cisco Packet Tracer, Cisco IOS
**Key Security Concept:** Network Segmentation & Least Privilege Access

## Architecture Topology
*(Note: View the `/Screenshots` folder for full topology images)*

### Design Decisions
Instead of a flat network, I segmented traffic into specific zones using a calculated VLSM (Variable Length Subnet Mask) scheme to minimize IP wastage and control broadcast domains.

| Zone | VLAN ID | Subnet | Purpose |
| :--- | :--- | :--- | :--- |
| **Inside** | 10 | `10.10.50.0/26` | Secured HR & Admin workstations. |
| **Guest** | 20 | `10.10.50.64/26` | Public Wi-Fi access (Restricted). |
| **DMZ** | N/A | `172.16.1.0/29` | Public-facing Web Server. |

## Implementation Details

### 1. VLAN & Trunk Configuration
I configured the Core Switch (`DXB_Switch`) to tag traffic using 802.1Q encapsulation. The trunk link to the router allows multiple VLANs to traverse a single physical cable.

**Evidence:** See `Configs/switch_final_config.txt` for the full port configuration.

### 2. Router-on-a-Stick & ACLs
The router (`DXB_Router`) acts as the gateway. I implemented an Extended ACL to act as a firewall rule.

**The Security Rule (ACL):**
The Guest network is allowed to access the Internet (Any) but explicitly DENIED access to the HR network (`10.10.50.0`).

```cisco
! From Configs/router_final_config.txt
ip access-list extended BLOCK_GUEST_TO_HR
 deny ip 10.10.50.64 0.0.0.63 10.10.50.0 0.0.0.63
 permit ip any any

Challenges & Troubleshooting
This section documents actual issues encountered during the build.

Issue: Native VLAN Mismatch

Problem: During the initial trunk configuration, I received %CDP-4-NATIVE_VLAN_MISMATCH errors between the switch and router.

Diagnosis: The router sub-interface was tagging frames, but the switch native VLAN settings were inconsistent.

Fix: I standardized the native VLAN across the trunk link to resolve the mismatch.

Issue: Ping Failure to DMZ

Problem: Initially, the Guest network could not reach the DMZ Web Server.

Fix: I realized I had not enabled the physical interface for the DMZ (G0/1) and assigned it the correct gateway IP 172.16.1.1.

Verification
I performed ping tests to verify the security rules:

Guest -> Internet: SUCCESS (Allowed)

Guest -> HR: FAILED (Blocked by ACL BLOCK_GUEST_TO_HR) - See /Screenshots for proof.
