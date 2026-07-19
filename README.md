# Proxmox Lab Environment

**A step-by-step guide to deploy a virtual networking lab environment in Proxmox.**

The documentation covers the deployment and basic configuration of both components, including networking and connectivity. The environment is intended for learning, testing, and validating Linux and networking concepts in an isolated setup.
This lab can serve as a foundation for more advanced networking and automation scenarios.


## 🎯 Lab Objectives
✅ Deploy a **Debian server VM** in Proxmox\
✅ Deploy a **vJunos switch VM** in Proxmox\
✅ Configure **LACP (802.3ad)** between Debian and vJunos\
✅ Test connectivity and validate the setup\
✅ (Optional) Extend with **VLANs, LLDP, and more**\

---
```
+------------------------------------------------------------------+
|                                                                  |
|                           Proxmox VE                             |
|                           Hypervisor                             |
|                                                                  |
|       +---------------+               +---------------+          |
|		|               |-----vmbr10----|               |          |
|       | Debian 13 VM  |      LACP     |   vJunos VM   |          |
|		|               |-----vmbr11----|               |          |
|       +---------------+               +---------------+          |
|               |                               |                  |
|               +-------------vmbr0-------------+                  |
|                               |                                  |
--------------------------------+----------------------------------+
                                |
                         Physical Network
```                         


