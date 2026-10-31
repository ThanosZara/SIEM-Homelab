## Home Lab with SIEM Logging and NIDS

An isolated home lab built inside VirtualBox, using pfSense, Snort NIDS, and Splunk SIEM to simulate live network traffic, log analysis and intrusion detection. This project was built to demonstrate core SOC analyst skills like alert triaging, log correlation and real-time event monitoring.

## Overview

This project simulates a small SOC environment running entirely on a single host machine using Oracle VirtualBox.

Network traffic, security events, and endpoint telemetry across the isolated internal network are inspected by a Network Intrusion Detection System (NIDS) and forwarded to a central SIEM platform.

This lab showcases practical skills in:

- Log collection, parsing, and SPL query construction
- Alert triage and initial incident verification
- Correlating network sensor alerts with endpoint activity
- Simulating adversary techniques and validating defensive visibility

## Lab Architecture

<img width="816" height="765" alt="εικόνα" src="https://github.com/user-attachments/assets/42c564b0-b138-469e-ab3c-864f0636f20b" />

	
The lab is built around a VirtualBox network that isolates the virtual machines from the physical home network.

Environment:

- **Host Machine**: Workstation providing resources(RAM, storage) for all virtual machines. 
- **pfSense Virtual Firewall**: Serves as the gateway for the internal network,  managing routing and traffic isolation.
- **Splunk SIEM VM**: Linux mint system running Splunk Enterprise for centralized log ingestion, indexing and dashboard monitoring.
- **Snort NIDS VM**: Ubuntu Server running Snort in promiscuous mode to inspect LAN traffic and output alerts.
- **Kali Linux VM**: Offensive security machine used for executing attack simulations.
- **Windows 10 VM**: Victim endpoint configured to forward security event logs to the SIEM.

## Implementation Stages

#### 1. Network Gateway & Firewall Setup

- Deployed pfSense Virtual Firewall in VirtualBox to serve as the gateway (`172.30.2.1`).
- Configured dual network adapters to link WAN connectivity with the internal LAN.
- Created and isolated the internal network subnet (`172.30.2.0/24`).
- Verified gateway routing, DHCP scope, and firewall rule enforcement.
- Configured native Syslog logging to output firewall traffic logs over UDP 514 to Splunk.

#### 2. Splunk SIEM Deployment (Linux Mint)

- Installed Linux Mint (`172.30.2.104`) on the internal network as the central SIEM host.
- Installed and configured Splunk Enterprise.
- Configured receiving inputs for Splunk Universal Forwarder traffic (TCP Port 9997) and Syslog streams (UDP Port 514).
- Verified active indexer inputs and web dashboard access.

#### 3. Snort NIDS Deployment (Ubuntu Server)

- Provisioned Ubuntu Server (`172.30.2.110`) on the internal network.
- Configured Snort NIDS to inspect network traffic on `172.30.2.0/24`.
- Updated and configured rulesets to generate signature-based alerts.
- Installed and configured Splunk Universal Forwarder (UF) to monitor and ship NIDS alerts to Splunk over TCP Port 9997.

#### 4. Windows 10 Endpoint Configuration

- Deployed Windows 10 victim host (`172.30.2.103`) on the internal LAN.
- Installed Sysmon with custom configuration to log process creation and network events.
- Installed and configured Splunk Universal Forwarder (UF) to ship Windows Event Logs and Sysmon logs over TCP Port 9997.
- Verified active endpoint log ingestion inside Splunk.

#### 5. Kali Linux & Attack Validation

- Deployed Kali Linux (`172.30.2.108`) onto the internal network as the offensive node.
- Executed targeted reconnaissance scans (`nmap`) and endpoint attack simulations across multiple internal hosts.
- Confirmed real-time log ingestion in Splunk across all active forwarders and syslog streams (pfSense Syslog, Snort UF, and Windows UF).
- Constructed custom SPL queries to correlate network alert triggers with host process activity.

## Detection Scenarios & Threat Hunting

#### Scenario 01: Network Reconnaissance

- Primary Tactic (MITRE): Reconnaissance (T1046)
- Main Log Source: Snort NIDS 
- Detailed Documentation: docs/01-nmap-recon.md

