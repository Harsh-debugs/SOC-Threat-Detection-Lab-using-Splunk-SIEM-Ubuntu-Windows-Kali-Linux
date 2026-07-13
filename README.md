# SOC Threat Detection and Monitoring using Splunk SIEM and Windows Sysmon

A small-scale Security Operations Center (SOC) lab built to detect and
visualize authentication attacks, reconnaissance activity, and process
execution on a monitored Windows endpoint — using Splunk Enterprise as the
SIEM platform and Microsoft Sysmon for deep endpoint telemetry.

Built as part of a cybersecurity internship at **Tata Consultancy Services, Hyderabad**.

---

## Overview

Security teams can't detect what they can't see. This project centralizes
logs from a Windows endpoint into a Splunk SIEM, simulates real attack
patterns (brute-force login, network reconnaissance) against that endpoint,
and confirms the SIEM can detect them — through dashboards, saved searches,
and scheduled alerts.

**What it demonstrates:**
- End-to-end log pipeline: endpoint → forwarder → SIEM indexer
- SPL (Search Processing Language) query writing for real detection use cases
- Dashboard and alert design for SOC-style monitoring
- Attack simulation and validation using industry-standard offensive tools

---

## Lab Architecture

Three virtual machines were built in Oracle VirtualBox, connected via a
shared NAT network (`192.168.56.0/24`):

| VM | OS / Role | Responsibilities |
|---|---|---|
| VM 1 | Ubuntu Linux — Splunk Enterprise Server | Receive forwarded logs, index, search, dashboards, alerts |
| VM 2 | Windows 11 — Victim Machine | Universal Forwarder + Sysmon installed; generates Security & Sysmon logs |
| VM 3 | Kali Linux — Attacker Machine | Nmap reconnaissance, Hydra brute-force attempts |

![Lab Architecture](screenshots/lab-network-architecture.png)

---

## Tools Used

| Category | Tools |
|---|---|
| SIEM | Splunk Enterprise, Splunk Universal Forwarder |
| Endpoint Telemetry | Windows Security Event Log, Microsoft Sysmon |
| Attack Simulation | Nmap, Hydra |
| Virtualization | Oracle VirtualBox |
| OS | Ubuntu Linux, Windows 11, Kali Linux |

---

## What Was Detected

| Detection | Source | Event ID |
|---|---|---|
| Failed logon attempts | Windows Security Log | 4625 |
| Successful logons | Windows Security Log | 4624 |
| Process creation | Sysmon | 1 |
| DNS queries | Sysmon | 22 |

Three scheduled alerts were configured in Splunk (hourly, triggering on
`Number of Results > 0`):
- **Multiple Failed Logon Detection**
- **Successful Logon Detection**
- **DNS Query Activity**

All three were verified to trigger correctly during testing. See
[`splunk/alerts.md`](splunk/alerts.md) for full configuration details.

---

## Attack Simulation

From the Kali Linux attacker machine:
- **Reconnaissance:** `nmap -Pn -n -p 445 <Windows_IP>` — identified the exposed SMB service.
- **Brute-force authentication:** Hydra against SMB, generating repeated failed-logon events.
- Manual failed/successful login attempts and DNS/process-execution commands were also run directly on the Windows endpoint to generate baseline telemetry.

Full command list and expected detection outcomes: [`attack-simulation/commands.md`](attack-simulation/commands.md)

---

## Dashboard

A single **SOC Threat Detection Dashboard** was built in Splunk, consolidating:
- Total Process Creation Events
- Failed Logons / Successful Logons (live counts)
- DNS Queries
- Top Executed Processes (bar chart)
- Recent Failed Logons (table)

![SOC Dashboard](screenshots/dashboard.png)

Full dashboard source: [`splunk/dashboard-source.xml`](splunk/dashboard-source.xml)

---

## SPL Queries

All detection and dashboard-panel queries are documented in
[`splunk/spl-queries.md`](splunk/spl-queries.md), including:
- Failed/successful logon searches
- Process creation and top-executed-process searches
- DNS query extraction
- Event-count queries used to power dashboard panels

---

## Testing / Verification

Every simulated attack was treated as a discrete test case and manually
verified against the expected Splunk behaviour:

| Test | Action | Detected Via |
|---|---|---|
| T01 | Nmap SMB scan | Network/connection events |
| T02 | Hydra brute-force | Event ID 4625 |
| T03 | Manual failed login | Event ID 4625 |
| T04 | Manual successful login | Event ID 4624 |
| T05 | DNS query (`nslookup`) | Sysmon Event ID 22 |
| T06 | Windows command execution | Sysmon Event ID 1 |

All six test cases passed, and all three alerts triggered correctly.

---

## Repository Structure

```
soc-threat-detection-splunk/
├── README.md
├── splunk/
│   ├── spl-queries.md
│   ├── dashboard-source.xml
│   └── alerts.md
├── attack-simulation/
│   └── commands.md
├── sysmon/
│   └── sysmonconfig.xml
└── screenshots/
    ├── lab-network-architecture.png
    ├── dashboard.png
    ├── failed-logons.png
    ├── successful-logons.png
    ├── process-creation.png
    ├── dns-queries.png
    ├── nmap-output.png
    ├── hydra-output.png
    └── triggered-alerts.png
```

---

## Future Scope

- Integrate Active Directory for domain-level authentication monitoring
- Map detected events to MITRE ATT&CK techniques
- Incorporate external threat-intelligence feeds
- Add SOAR-based automated response
- Extend to Splunk Enterprise Security for advanced correlation

---

## Author

**Harsh Chourasia**
B.Tech, Electronics & Computer Science, KIIT Deemed to be University
Cybersecurity Intern, Tata Consultancy Services, Hyderabad
