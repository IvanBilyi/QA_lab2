# Network Environment Setup: DHCP Server & ARP Protocol Analysis

**Course:** Software & Hardware QA for Embedded Systems  
**Author:** Ivan Bilyi, Group IO-34 (Variant 2)  
**Institution:** Igor Sikorsky Kyiv Polytechnic Institute (KPI), Faculty of Informatics and Computer Engineering  
**Year:** 2026  

---

## Project Overview
This repository contains the deliverables for Laboratory Work #2 (3) focused on layer 2 (Data Link) and layer 3 (Network) protocol analysis within the OSI model. 

The main objectives include:
* Manual ARP table manipulation and static cache validation.
* Deployment and configuration of `isc-dhcp-server` on Ubuntu 22.04 LTS.
* Full trace analysis of the DHCP **DORA** process (`DISCOVER`, `OFFER`, `REQUEST`, `ACK`).
* Simulation and QA documentation (Defect Report & Test Case) of a **Rogue DHCP conflict** (Race Condition) in a single-node setup.

---

## Repository Structure
* `Лабораторна робота №2 Білий Іван ІО-34.pdf` — Complete laboratory report including execution steps, embedded terminal output with timestamps/identity verification, and QA documentation.
* `dhcpd.conf` — ISC DHCP server configuration file defining the subnet parameters (`10.0.2.0/24`) and dynamic address lease range (`10.0.2.50` – `10.0.2.100`).
* `dhcp_conflict.pcapng` — Wireshark network capture file demonstrating the Rogue DHCP conflict (multiple `DHCPOFFER` frames responding to a single `DHCPDISCOVER`).

---

## Tech Stack & Environment
* **OS:** Ubuntu 22.04 LTS (VirtualBox VM)
* **DHCP Service:** `isc-dhcp-server` (ISC DHCP v4.4.1)
* **Traffic Analyzer:** Wireshark
* **Networking Tools:** `iproute2` (`ip neigh`), `dhclient`, `ping`

---

## Summary of Test Results

### 1. ARP Protocol Analysis (Variant 2)
* **Objective:** Validate network stack behavior when utilizing a static (`PERMANENT`) ARP table entry.
* **Execution:** Deleted dynamic ARP entry for gateway `10.0.2.2` and assigned a static entry via `ip neigh replace 10.0.2.2 lladdr <MAC> dev enp0s3 nud permanent`.
* **Result:** Captured traffic during ICMP ping (`ping -c 4 10.0.2.2`) confirmed **zero broadcast `ARP Request` frames**, proving that static caching bypasses hardware address resolution.

### 2. DHCP Server Deployment & DORA Cycle
* **Execution:** Configured `/etc/default/isc-dhcp-server` (bound to `enp0s3`) and `/etc/dhcp/dhcpd.conf`. Verified active status via `systemctl status isc-dhcp-server`.
* **Result:** Triggered interface lease renewal (`sudo dhclient -v enp0s3`) and tracked active leases in `/var/lib/dhcp/dhcpd.leases`.

### 3. Rogue DHCP Conflict Simulation (QA Defect Report)
* **Objective:** Simulate a dual-DHCP server conflict on a single host.
* **Execution:** Executed `dhclient` with both VirtualBox NAT DHCP (`10.0.2.2`) and local `isc-dhcp-server` (`10.0.2.15`) running simultaneously.
* **Result:** Wireshark captured two conflicting `DHCPOFFER` frames for transaction `0x59f66e05`. Documented as a critical network vulnerability (`BUG-DHCP-CONFLICT-01`) describing the Race Condition behavior.
