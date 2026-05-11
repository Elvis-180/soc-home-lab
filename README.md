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

## Lesson Learned
-	Placement matters more than tools: IDS effectiveness depends on correct network positioning; poor placement creates blind spots regardless of configuration quality.
- SIEM visibility is not automatic: Log ingestion must be deliberately designed (forwarders, indexes, and parsing). If not, data exists but is invisible.
- Single data sources are insufficient: Firewall logs alone cannot explain incidents. Effective investigation requires correlation across firewall, IDS, and SIEM data.
- Correlation is the real skill: Security monitoring is not about tools, it’s about connecting signals across multiple layers to reconstruct events accurately.



## Conclusion
This architecture demonstrates how layered security tools work together to provide visibility, detection, and analysis of network traffic in a simulated SOC environment.
