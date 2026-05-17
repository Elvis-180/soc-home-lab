# SOC Home Lab For Network Threat Detection

## Overview
This lab simulates a segmented enterprise network designed to demonstrate traffic monitoring, intrusion detection, and centralized log analysis using IDS and SIEM tools.

The environment focuses on observing how traffic flows across different network zones and how security tools interpret and analyze that traffic.

---

## Objectives
- Design a segmented SOC-style network
- Map detected behaviour to MITRE ATT&CK techniques for threat context
- Simulate real-world cyber attacks from Kali Linux
- Monitor traffic and Detect attacks using Snort IDS and Sysmon endpoint monitoring
- Forward all logs to Splunk for centralized analysis
- Build correlation rules and dashboards in Splunk
- Practice incident response and threat hunting

  ---

## Lab Architecture Design

### Network Segmentation
Assigned pfsense 3 interface (WAN, LAN and OPT1)
- External OPT1 Network: [Kali Linux]
- Internal LAN Network: [Windows Server, Windows 10]
- Management / Monitoring Network: [Splunk (SIEM), Snort (IDS), Sysmon, Pfsense Logs]

---

## Architecture Diagram
             [ Kali Linux — 192.168.2.5/24 . OPT1 ]
     Attacker · Nmap · Metasploit . Hydra
              │
              │  recon · brute force · exploit
              ▼
    ┌─────────────────────────────┐
    │          pfSense            │   
    │        Firewall             │
    └──────────────┬──────────────┘
               │
               │  192.168.1.0/24 LAN
               │
    ┌──────────────▼──────────────┐
    │       Ubuntu Server         │   
    │  Snort IDS · Splunk UF      |
    │        192.168.1.9          |
    └──────────────┬──────────────┘
               │
       ┌───────┴────────┐
       │                │
    ┌──────▼──────┐  ┌──────▼──────┐
    │ Windows     │  │ Windows 10  │   
    │ Server 2022 │  │  Domain PC  │
    │ 192.168.1.1 │  │     UF      │
    │ AD DC · DNS │◄─┤ 192.168.1.3 │
    │ Sysmon · UF │  │             │
    └─────────────┘  └─────────────┘
               │
               │  all logs forwarded
               ▼
    ┌─────────────────────────────┐
    │     Splunk Enterprise       │   
    │  SIEM / Correltion          |
    │      192.168.1.10:8000      |
    └─────────────────────────────┘                         
         
---

## Lab Setup
### Step 1  Install VirtualBox
1.	Downloaded VirtualBox from https://www.virtualbox.org 
2.	Run the installer and follow the prompts — accept all defaults.
3.	Also install the VirtualBox Extension Pack from the same download page (adds USB 3.0, RDP).
4.  Virtualisation is already enabled (enabled in the BIOS)

## Step 2  Install & Configure pfSense
pfSense is  firewall. It has three virtual network interfaces:
•	Interface 1 (em0 / WAN side)
•	Interface 2 (em1 / LAN side): Connected to the external machine ( Kali Linux)
. Interface 3 (em3/ OPT1 side): Connected to internal machines (Windows server, windows 10, Ubuntu Server)

### 2a  Create the pfSense VM
1.	In VirtualBox → New
2.	Name: pfSense, Type: BSD, Version: FreeBSD (64-bit)
3.	RAM: 1024 MB, Disk: 30 GB
4.	Before starting, go to Settings → Network
-	Adapter 2: Internal Network → Name: attacker-net
-	Adapter 3: Internal Network → Name: internal-net
  
### 2b Install pfSense
1.	Boot from the pfSense ISO
2.	Accept defaults through the installer 
3.	Reboot after install, remove ISO
   
### 2c  Configure Interfaces
After reboot, at the console:
1.	Select option 1 — Assign Interfaces
- OPT1 → em3 (attacker-net side)
-	LAN → em1 (internal-net side)
2.	Select option 2 — Set Interface IP Addresses
-	 WAN IP: 10.0.2.15/24
-	 LAN IP: 192.168.1.2/24 (internal subnet)
-  OPT1 IP: 192.168.2.1/24 (external subnet)

### 2d Access the pfSense Web UI
- From any internal machine browser: http://192.168.20.1
- Default login: admin / pfsense
- The Password was changed immediately

## Step 3  Install Ubuntu Server + Snort IDS
### 3a  Create the Ubuntu VM
1.	VirtualBox → New
2.	Name: Ubuntu-IDS, Type: Linux, Version: Ubuntu (64-bit)
3.	RAM: 2048 MB, Disk: 50 GB
4.	Settings → Network → Adapter 1: Internal Network → internal-net
   
### 3b Install Ubuntu Server
1.	Boot from Ubuntu Server ISO
2.	Follow installer: set hostname and password
3.	Set IP (192.168.1.9)
4.	After install, log in and update: sudo apt update && sudo apt upgrade -y

## Step 4  Install Windows Server & Windows 10
Windows Server VM
1.	VirtualBox → New
2.	Name: WinServer, Type: Microsoft Windows, Version: Windows 2025 (64-bit)
3.	RAM: 4096 MB, Disk: 50 GB
4.	Network → Adapter 1: Internal Network → internal-net
5.	Boot from Windows Server ISO, follow setup wizard
6.	Set a strong Administrator password
  
Windows 10 VM
1.	Repeat above steps, Version: Windows 10 (64-bit)
2.	RAM: 2048 MB, Disk: 40 GB
3.	Network → Adapter 1: Internal Network → internal-net

## Step 5 — Install Sysmon on Windows server
Sysmon (System Monitor) logs detailed process, network, and file activity.
On  Windows Server
1.	Download Sysmon from Microsoft Sysinternals:
https://learn.microsoft.com/sysinternals/downloads/sysmon
2.	Download a community Sysmon config (recommended — SwiftOnSecurity):
https://github.com/SwiftOnSecurity/sysmon-config
3.	Open PowerShell as Administrator and run:
   
**Navigate to your download folder**
- cd C:\Users\Admin\Downloads

**Install Sysmon with the config file**
- .\Sysmon64.exe -accepteula -i sysmonconfig-export.xml

**Verify Sysmon is running**
- Get-Service Sysmon64

**View Sysmon Logs**
- Open Event Viewer, Application and Services Logs, Microsoft, Windows, Symon, Operational

## Step 6  Install Kali Linux

6a  Create the Kali VM
1.	VirtualBox → New
2.	Name: Kali-Attacker, Type: Linux, Version: Debian (64-bit)
3.	RAM: 2048 MB, Disk: 50 GB
4.	Network → Adapter 1: Internal Network → attacker-net
   
6b  Install Kali
1.	Boot from Kali ISO
2.	Select Graphical Install, follow prompts
3.	Set hostname (tem), password ()
4.	Set IP (192.168.2.5)
5.	After install, update:
- sudo apt update && sudo apt upgrade -y

## Step 7 Test Connectivity 
- Test connectivity between each machine using command (Ping)

## Step 8 Install and Configure Snort IDS
1.	Go to https://www.snort.org → create a free account
2.	After logging in, go to your profile → copy your Oinkcode
3.	Install Snort on Ubuntu via apt

## Step 10 — Install Splunk Universal Forwarder
The Universal Forwarder ships logs from Windows/Ubuntu to a Splunk SIEM instance.
On Windows Server / Windows 10:
1.	Download from: https://www.splunk.com/en_us/download/universal-forwarder.html
2.	Run the installer, set:
•	Receiving Indexer: IP of your Splunk server, port 9997
3.	After install, configure what to monitor:
Open C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf and add:
- [WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
index = main

**On Ubuntu**
Download and install 
- wget -O splunkforwarder.deb "https://download.splunk.com/products/universalforwarder/..."
- sudo dpkg -i splunkforwarder.deb

**Start and configure**
- sudo /opt/splunkforwarder/bin/splunk start --accept-license
- sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.20.x:9997
- sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/snort/alert

---

##  Tools & Roles

###  Attack Machine
| Tool | Type | Role |
|------|------|------|
| **Kali Linux** | Attacker OS | Simulates real-world attacks — port scanning, brute force, exploitation |

---

###  Network Security
| Tool | Type | Role |
|------|------|------|
| **pfSense** | Firewall / Router | Controls traffic, blocks/allows connections, logs network activity |

---

###  Endpoints (Target Machines)
| Tool | Type | Role |
|------|------|------|
| **Windows Server (Domain Controller)** | Endpoint | Active Directory domain controller, primary attack target |
| **Windows 10** | Endpoint | Domain Computer, simulates a real user machine |
| **Sysmon** | Endpoint Monitor | Logs process creation, network connections, file & registry changes on Windows machines |
| **Splunk Universal Forwarder** | Log Shipper | Ships Sysmon & Windows Event logs to Splunk |

---

###  Network Detection
| Tool | Type | Role |
|------|------|------|
| **Ubuntu Server** | Host OS | Hosts Snort IDS |
| **Snort** | Network IDS | Monitors network traffic, detects suspicious patterns and known attack signatures |

---

###  SIEM
| Tool | Type | Role |
|------|------|------|
| **Splunk Enterprise** | SIEM | Collects, indexes, correlates and visualizes all logs from every source in the lab |

---

## Traffic Flow Description
- Kali sends malicious traffic into the network. That traffic hits pfSense first, which enforces firewall rules and logs everything passing through.
- Traffic then enters the LAN where Ubuntu/Snort is listening in promiscuous mode  it inspects all traffic on the segment and generates alerts on anything suspicious.
- The internal machines, Windows Server and Windows 10, communicate normally  the workstation authenticates against the Domain Controller for login, DNS, and Group Policy.
- All machines forward their logs to Splunk. pfSense sends firewall logs, Snort sends IDS alerts, and the Windows machines send Sysmon and event logs. Splunk ties it all together so you have full visibility across the entire network from a single place.

---

## Key Observations
- Firewall sees traffic flow, not deep packet details
- IDS detects packet-level anomalies
- SIEM provides aggregated visibility across sources

---
## Screenshot
- Vitual Machine
  <img width="1920" height="1080" alt="Screenshot 2026-05-07 125102" src="https://github.com/user-attachments/assets/5cc60bc2-e12e-423f-a53c-fa475e93278e" />
- Logs ingested into Splunk (SIEM)
  <img width="1920" height="1080" alt="Screenshot 2026-05-07 131246" src="https://github.com/user-attachments/assets/491e6f96-626d-4f7c-927a-69d621a5b174" />

## Lesson Learned
-	Placement matters more than tools: IDS effectiveness depends on correct network positioning; poor placement creates blind spots regardless of configuration quality.
- SIEM visibility is not automatic: Log ingestion must be deliberately designed (forwarders, indexes, and parsing). If not, data exists but is invisible.
- Single data sources are insufficient: Firewall logs alone cannot explain incidents. Effective investigation requires correlation across firewall, IDS, and SIEM data.
- Correlation is the real skill: Security monitoring is not about tools, it’s about connecting signals across multiple layers to reconstruct events accurately.



## Conclusion
This architecture demonstrates how layered security tools work together to provide visibility, detection, and analysis of network traffic in a simulated SOC environment.
