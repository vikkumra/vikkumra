# Home Network Audit Lab

## Overview
I set up a Kali Linux virtual machine and used it to practice basic network recon techniques. Beggining in a isolated lab network then against my own personal home network. (i redacted my personal information using 'x')

- **Hypervisor:** VMware Fusion Pro (Apple Silicon)
- **Guest OS:** Kali Linux 2026.2 (ARM64)
- **Host:** MacBook (M3, 8GB RAM)


## Method

### 1. Host Discovery (NAT network)
Initially i ran a ping sweep in Fusion's default NAT mode to confirm basic Nmap functionality: nmap -sn 192.xxx.xx.0/24

This revealed the isolated virtual network Fusion created by default.

### 2. Switching to Bridged Mode
To scan real devices on my network, I switched the VM's network adapter from NAT to 'Bridged'. this gave Kali its own IP address directly from my home router rather than sitting behind a private virtual network. 

### 3. Host Discovery (home network)
I then ran another ping sweep: nmap -sn 192.xxx.x.0/24

This performs an ARP-based ping sweep across the local subnet. Because ARP (Address Resolution Protocol) operates at Layer 2, it reveals live hosts even when devices block ICMP ping. The scan identified 20 live devices, each returning a MAC address whose vendor prefix (OUI) could be matched to a manufacturer.

### 4. Service & Version Detection
I Selected the router as the most interesting target and ran: nmap -sV 192.xxx.x.x

This performed a full port scan.

## Findings

| Port | State | Service | Notes |
|------|-------|---------|-------|
| 22 (SSH) | filtered | — |
| 23 (Telnet) | filtered | — |
| 53 (DNS) | open | dnsmasq 2.83 |
| 80 (HTTP) | open | "Xfinity Broadband Router Server" |
| 111 (RPC) | filtered | — |
| 443 (HTTPS) | open | "Xfinity Broadband Router Server" |
| 8080 / 8181 / 9000 | filtered | — |

### Key observation
The router's web server identifies itself as an **"Xfinity Broadband Router Server"**, despite being supplied by a different UK ISP. This strongly suggests the hardware/firmware is **whitelabeled** — manufactured by a single vendor and rebranded for multiple ISPs, with the original server signature left unchanged in the HTTP headers. This kind of inconsistency can be a useful reconnaissance signal in real assessments, since it can reveal the true underlying hardware/software platform even when a device has been rebranded.

The HTTP response headers also showed several security-conscious configurations already in place, including `Content-Security-Policy`, `X-Frame-Options: deny`, and `Strict-Transport-Security` — indicating the router's admin interface has reasonable baseline web hardening.

## What I learned
- The practical difference between NAT and bridged VM networking, and when each is appropriate
- How ARP-based discovery works and why it's more reliable than ICMP ping on a LAN
- How to interpret Nmap's open/closed/filtered port states
- How service banners and HTTP headers can reveal information about a device's true origin, even when rebranded
- The importance of not publishing raw scan data (MAC addresses, hostnames) publicly, even from one's own network

## Next steps
- Set up a deliberately vulnerable target VM (e.g., Metasploitable2) on an isolated NAT network to safely practise deeper enumeration and basic exploitation techniques
- Begin documenting TryHackMe room write-ups alongside this lab work
