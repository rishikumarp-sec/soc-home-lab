# SOC Home Lab — Splunk SIEM & Threat Detection

## Overview
A fully functional SOC analyst home lab built on Mac M1 using
UTM virtualization. Simulates real-world attack and detection
scenarios using industry-standard tools.

## Architecture
| Machine | Role | IP |
|---------|------|----|
| Mac M1 | Analyst — Splunk SIEM | 192.168.64.1 |
| Ubuntu Server | Victim endpoint | 192.168.64.21 |
| Kali Linux | Attacker machine | 192.168.64.14 |

## Tools Used
- Splunk Enterprise (SIEM)
- Splunk Universal Forwarder
- Kali Linux (Hydra, Nmap, Nikto)
- Ubuntu Server 22.04 ARM64
- UTM virtualization on Apple Silicon

## Log Sources Monitored
- /var/log/auth.log — SSH authentication events
- /var/log/syslog — system events
- /var/log/apache2/access.log — web traffic
- /var/log/apache2/error.log — web errors

## Attacks Simulated
- SSH brute force via Hydra
- Web vulnerability scanning via Nikto
- Port scanning via Nmap
- Privilege escalation attempts

## Skills Demonstrated
- Splunk Enterprise and SPL query writing
- SIEM configuration and log ingestion
- Network segmentation and virtualization
- SSH brute force detection
- Linux administration (Ubuntu and Kali)
- Incident detection and analysis
- Apple Silicon M1 lab setup

## Repository Structure
- setup/setup.md — full step by step lab setup guide
- splunk/spl-queries.md — all SPL detection queries
- reports/incident-report-001.md — first incident report
