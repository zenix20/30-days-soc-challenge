# Day 25: Capstone 2 : Set up a pfsense firewall

**Tools:** Proxmox, pfSense

## Scenario

A second infrastructure-focused capstone: deploy pfSense (an open-source firewall/router platform) inside Proxmox, positioned between a WAN-facing bridge and an isolated LAN bridge, with a client VM sitting behind it to verify the firewall is correctly routing and filtering traffic.

The lab architecture, per the guide:

```
Internet / Home Router
        |
      vmbr0 (WAN)
        |
   pfSense Firewall
        |
      vmbr1 (LAN)
        |
   Client VM / Lab VMs
```

## Questions & Approach

### 1. Which Proxmox bridge is used for the pfSense LAN interface in this lab?

The guide creates two bridges: vmbr0 (the existing default bridge, tied to the physical network interface, used as WAN) and a newly created vmbr1 (an isolated internal bridge with no physical interface attached, used as LAN). The pfSense VM's second network interface is explicitly attached to vmbr1.

**Answer: vmbr1**

### 2. What is the default gateway IP for LAN client VMs in this lab?

During LAN interface configuration, pfSense's LAN IP is set to 192.168.10.1/24, which becomes the gateway address every client VM on that isolated network receives via DHCP.

**Answer: 192.168.10.1**
