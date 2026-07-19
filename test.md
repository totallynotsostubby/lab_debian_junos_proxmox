# Lab: Debian Server and vJunos Switch in Proxmox with LACP

---
## 📋 Document Information
   **Item**            | **Value**                          |
 |---------------------|------------------------------------|
 | Category            | LAB                                |
 | Document ID         | LAB-001                            |
 | Version             | **2.0**                            |
 | Status              | **Final**                          |
 | Last Updated        | July 19, 2026                      |
 | Audience            | Network Engineers, System Administrators, DevOps, Home Lab Enthusiasts |
 | **Proxmox VE**      | **10.0** (Tested on 9.2.3+)        |
 | **Debian**          | **12.5 (Bookworm) amd64**          |
 | **vJunos**          | **23.4R1** (Publicly available)    |
 | Estimated Time      | 75–90 minutes                      |

---

## 📚 Table of Contents
1. [Objective](##-objective)
2. [Lab Architecture](##-lab-architecture)
3. [Prerequisites](#prerequisites)
4. [Preparations](#preparations)
   - [4.1 Downloads](#41-downloads)
   - [4.2 Upload Debian ISO to Proxmox](#42-upload-debian-iso-to-proxmox)
   - [4.3 Upload vJunos Image to Proxmox](#43-upload-vjunos-image-to-proxmox)
5. [Installations](#installations)
   - [5.1 Proxmox VE Assumptions](#51-proxmox-ve-assumptions)
   - [5.2 Debian VM](#52-debian-vm)
   - [5.3 vJunos VM](#53-vjunos-vm)
6. [Configurations](#configurations)
   - [6.1 Proxmox OVS Bridges](#61-proxmox-ovs-bridges)
   - [6.2 Debian Network Bonding (LACP)](#62-debian-network-bonding-lacp)
   - [6.3 vJunos LACP Configuration](#63-vjunos-lacp-configuration)
7. [Verification](#verification)
8. [Troubleshooting](#troubleshooting)
9. [Appendix](#appendix)
   - [9.1 Useful Commands](#91-useful-commands)
   - [9.2 VLAN Extension (Optional)](#92-vlan-extension-optional)

---

---

## Objective
This lab guide provides a **step-by-step** approach to deploy a **Debian server** and a **vJunos virtual switch** in **Proxmox VE**, with **Link Aggregation Control Protocol (LACP)** configured between them.

The environment serves as a **practical platform** for learning and testing:
- **LACP (802.3ad)** – Link aggregation for redundancy and bandwidth
- **LLDP** – Link Layer Discovery Protocol for topology discovery
- **VLANs** – Virtual Local Area Networks for segmentation
- **Virtual networking** in Proxmox VE

By the end of this lab, you will have a **fully functional virtual network** with:
✅ A **Debian VM** acting as a Linux host with LACP bonding
✅ A **vJunos VM** acting as a virtual switch with LACP
✅ **Redundant, high-bandwidth** connectivity between the two

---

## Lab Architecture

### Network Topology
The lab consists of **two virtual machines** connected via **three Proxmox bridges**:

```mermaid
graph TB
    subgraph Proxmox["Proxmox VE Hypervisor"]
        subgraph Debian["Debian VM\n(Host)"]
            D0[("eth0\nvmbr0\nManagement")]
            D1[("eth1\nvmbr10\nLACP Member 1")]
            D2[("eth2\nvmbr11\nLACP Member 2")]
            B0[("bond0\nLACP Aggregation")]
        end
        subgraph vJunos["vJunos VM\n(Switch)"]
            V0[("ge-0/0/0\nvmbr0\nManagement")]
            V1[("ge-0/0/1\nvmbr10\nLACP Member 1")]
            V2[("ge-0/0/2\nvmbr11\nLACP Member 2")]
            A0[("ae0\nLACP Aggregation")]
        end
        subgraph Bridges["OVS Bridges"]
            vmbr0[("vmbr0\nManagement")]
            vmbr10[("vmbr10\nLACP Link 1")]
            vmbr11[("vmbr11\nLACP Link 2")]
        end
    end
    subgraph PhysNet["Physical Network"]
        Router[("Router/Switch")]
    end

    D0 --> vmbr0
    V0 --> vmbr0
    vmbr0 --> Router

    D1 --> vmbr10
    D2 --> vmbr11
    V1 --> vmbr10
    V2 --> vmbr11

    B0 --> D1
    B0 --> D2
    A0 --> V1
    A0 --> V2
