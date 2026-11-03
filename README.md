## Home Lab with SIEM Logging and NIDS

An isolated home lab built inside VirtualBox, using pfSense, Snort NIDS, and Splunk SIEM to simulate live network traffic, log analysis and intrusion detection. This project was built to demonstrate core SOC analyst skills like alert triaging, log correlation and real-time event monitoring.

## Overview

This project simulates a small SOC environment running on a single host machine using Oracle VirtualBox.

Network traffic, security events, and endpoint telemetry across the isolated internal network are inspected by a Network Intrusion Detection System (NIDS) and forwarded to a central SIEM platform.

This lab showcases practical skills in:

- Log collection, parsing, and SPL query construction
- Snort rule writing and tuning
- Alert triage and initial incident verification
- Simulating adversary techniques and validating defensive visibility

## Lab Architecture

<img width="676" height="758" alt="εικόνα" src="https://github.com/user-attachments/assets/dc637618-37b2-4eb0-8b8b-672b7c69f511" />

	
The internal LAN is isolated from the physical home network. pfSense has a WAN adapter for outbound connectivity (for example, to install packages), while the attacker and victim VMs only reach other networks through pfSense.

#### Hosts


| Host        | OS                      | IP             | Role                                        | Agents / Services                  |
| ----------- | ----------------------- | -------------- | ------------------------------------------- | ---------------------------------- |
| pfSense     | pfSense `2.9.0`         | `172.30.2.1`   | Gateway, firewall, DHCP                     | Remote syslog to Splunk            |
| Splunk SIEM | Linux Mint `22.3`       | `172.30.2.104` | Log ingestion, search, alerting, dashboards | Splunk Enterprise                  |
| Snort NIDS  | Ubuntu Server `24.04.5` | `172.30.2.110` | Network intrusion detection                 | Snort, Splunk Universal Forwarder  |
| Windows 10  | Windows 10 `22H2`       | `172.30.2.103` | Victim endpoint                             | Sysmon, Splunk Universal Forwarder |
| Kali Linux  | Kali `2024.4`           | `172.30.2.108` | Attacker                                    | Nmap and other offensive tools     |
|             |                         |                |                                             |                                    |


#### VM resources

| VM          | vCPU | RAM   | Disk   |
| ----------- | ---- | ----- | ------ |
| pfSense     | `2`  | `2GB` | `25GB` |
| Splunk SIEM | `2`  | `3GB` | `50GB` |
| Snort NIDS  | `2`  | `3GB` | `25GB` |
| Windows 10  | `2`  | `3GB` | `25GB` |
| Kali Linux  | `2`  | `2GB` | `25GB` |

**Host Machine**: Intel Core i5-9400F (6 Cores / 6 Threads @ 2.90GHz), 16 GB RAM, ~1.3TB SSD, Windows 10(64-bit). 

## Implementation Stages

#### 1. Network Gateway & Firewall Setup

- Deployed pfSense in VirtualBox as the gateway (`172.30.2.1`) with two adapters: WAN and internal LAN.
- Created and isolated the internal subnet (`172.30.2.0/24`).
- Verified gateway routing, DHCP scope, and firewall rule enforcement.
- Enabled remote logging (Status → System Logs → Settings) to send firewall events to Splunk via syslog.

#### 2. Splunk SIEM Deployment (Linux Mint)

- Installed Linux Mint (`172.30.2.104`) on the internal network as the central SIEM host.
- Installed and configured Splunk Enterprise.
- Configured a receiving input for Universal Forwarders (TCP 9997) and a UDP input for pfSense syslog (port 5514).
-  Verified active inputs and web interface access.

#### 3. Snort NIDS Deployment (Ubuntu Server)

- Deployed Ubuntu Server (`172.30.2.110`) with Snort (`2.9.20`) inspecting traffic on `172.30.2.0/24`.
- **Traffic visibility:** The Snort VM's adapter in VirtualBox is set to **Promiscuous Mode: Allow All** (VM Settings → Network → Advanced), so it receives frames addressed to other VMs on the internal network.
- Wrote custom local rules (`rules/local.rules`), including a threshold-based port scan rule.
- Installed a Splunk Universal Forwarder to monitor the alert file and ship it to Splunk over TCP 9997.

#### 4. Windows 10 Endpoint Configuration

- Deployed Windows 10 victim host (`172.30.2.103`) on the internal LAN.
- Installed Sysmon using **Olaf Hartong's** `sysmon-modular` configuration to log process creation, network connections, and other endpoint events.
- Installed a Splunk Universal Forwarder to ship Windows Event Logs and Sysmon logs over TCP 9997.
- Verified ingestion in Splunk.

#### 5. Kali Linux & Attack Validation

- Deployed Kali Linux (`172.30.2.108`) as the offensive node.
- Ran Nmap scans against the internal network and validated that each log source captured the activity.
- Confirmed real-time ingestion across all three sources (pfSense syslog, Snort via UF, Windows via UF).

## Detection Scenarios

#### Scenario 01: TCP SYN port scan

- Primary Tactic (MITRE): Discovery (T1046)
- Main Log Source: Snort NIDS 
- Detailed Documentation: docs/01-nmap-recon.md

