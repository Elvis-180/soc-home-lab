# SOC Home Lab For Network Threat Detection

## Overview
This lab simulates a segmented enterprise network designed to demonstrate traffic monitoring, intrusion detection, and centralized log analysis using IDS and SIEM tools.

The environment focuses on observing how traffic flows across different network zones and how security tools interpret and analyze that traffic.

---

## Objectives
- Design a segmented SOC-style network
- Monitor network traffic and detect intrusions using IDS and firewall logging
- Centrlize and correlate security logs using a SIEM platefrom
- Simulate real-world attacks from an external adversary
- Map detected behaviour to MITRE ATT&CK techniques for threat context
- Understand visibility differences across network layers

  ---

## Lab Architecture Design

### Network Segmentation
Assigned pfsense 3 interface (WAN, LAN and OPT1)
- External OPT1 Network: [Kali Linux]
- Internal LAN Network: [Windows Server, Windows 10]
- Management / Monitoring Network: [Splunk (SIEM), Snort (IDS), Pfsense Logs]

---

## Architecture Diagram
                [ Kali Linux (OPT1) ]
                       |
                       | (Attacks: scan, brute force, exploit)
                       v
        -----------------------------------
        |            pfSense              |
        | Firewall + Traffic Logging     |
        -----------------------------------
            |                      |
            |                      |
     Internal LAN           Logging Forward
            |                      |
    ------------------             |
    |                |             |
    [Windows Server ]   [Windows 10]      |
        \            /            |
         \          /             |
          [Ubuntu + Snort IDS] ---|
                 |
                 v
        ---------------------
        | Splunk Enterprise |
        | SIEM / Correlation |
        ---------------------

### Components

#### Firewall / Gateway
- Tool: pfSense
- Role: Network routing, segmentation, and traffic control

#### Intrusion Detection System (IDS)
- Tool: Snort (Ubuntu Server)
- Role: Packet inspection and alert generation for suspicious traffic

#### SIEM Platform
- Tool: Splunk
- Role: Log ingestion, correlation, and event analysis

#### End Systems
- Windows Server (Domain Controller
- Windows 10 (Domain Computer)

---

## Traffic Flow Description
1. Traffic is generated from the Windows system
2. Traffic passes through pfSense firewall
3. Snort inspects packets for anomalies or rule matches
4. Logs are forwarded to Splunk for centralized analysis
5. Events are correlated and visualized in Splunk dashboards

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
