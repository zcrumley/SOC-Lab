# SOC Analyst Home Lab

## Overview

This project documents the design, setup, and use of a home SOC lab built to practice blue-team security monitoring, endpoint telemetry collection, log analysis, and incident investigation.

The lab uses Wazuh as the SIEM/XDR platform, a Windows 11 endpoint with Sysmon for detailed event logging, and a Kali Linux attack machine to generate controlled security events. 

## Objectives

- Build a functional SOC-style monitoring environment
- Collect and analyze endpoint security logs
- Configure Wazuh agents on Windows systems
- Use Sysmon to improve Windows telemetry
- Simulate common attack behaviors in a controlled lab
- Investigate alerts using a structured incident response process
- Document findings in a way that reflects real-world SOC workflows

## Lab Architecture

| Component | Purpose |
|---|---|
| Ubuntu Server VM | Hosts Wazuh manager and dashboard |
| Windows 11 VM | Monitored endpoint/victim machine |
| Kali Linux VM | Attack simulation machine |
| Wazuh | SIEM/XDR platform for alerting and log analysis |
| Sysmon | Windows telemetry and process/event monitoring |

## Skills Demonstrated

- SIEM monitoring
- Endpoint detection and response concepts
- Windows event log analysis
- Sysmon configuration
- Wazuh agent deployment
- Alert triage
- Incident investigation
- Basic threat simulation
- Network and host-based security monitoring
- Technical documentation

## Detection Scenarios

| Scenario | Status |
|---|---|
| Windows agent connected to Wazuh | Complete |
| Sysmon installed and forwarding logs | Complete |
| Failed login monitoring | Complete |
| PowerShell activity detection | Complete |
| Network scan detection | Planned |
| Brute-force simulation | Planned |
| Malware test file detection using EICAR | Planned |

## Screenshots

Screenshots are included throughout the documentation to show:

- Wazuh dashboard access
- Connected agents
- Generated alerts
- Event log details
- Investigation workflow

## Lessons Learned

This lab helped reinforce how endpoint telemetry, SIEM alerting, and structured investigation workflows come together in a SOC environment. The project also provided hands-on experience with troubleshooting agents, validating log sources, and documenting alerts in a professional format.
