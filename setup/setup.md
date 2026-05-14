# Home Lab Setup Guide

## Prerequisites
- Mac with Apple Silicon M1/M2/M3
- Minimum 16GB RAM recommended
- Minimum 100GB free disk space
- UTM virtualization app from mac.getutm.app
- Splunk account (free) from splunk.com

## Network Reference
| Machine | Role | IP |
|---------|------|----|
| Mac M1 | Analyst / Splunk | 192.168.64.1 |
| Ubuntu Server | Victim | 192.168.64.21 |
| Kali Linux | Attacker | 192.168.64.14 |

## Phase 1 — Splunk on Mac

Install Rosetta 2

    softwareupdate --install-rosetta --agree-to-license

Add to PATH and start Splunk

    echo 'export PATH=$PATH:/Applications/Splunk/bin' >> ~/.zshrc
    source ~/.zshrc
    splunk start --accept-license

Configure Splunk
- Open http://localhost:8000
- Settings → Licensing → Switch to Free License
- Settings → Forwarding and Receiving → Configure Receiving → Add port 9997

Enable auto-start

    sudo /Applications/Splunk/bin/splunk enable boot-start

## Phase 2 — Ubuntu Victim VM in UTM

UTM Settings
- Mode: Virtualize
- Architecture: ARM64
- RAM: 4096 MB
- Storage: 20 GB
- Network: Shared Network

Post install setup

    sudo apt update && sudo apt upgrade -y
    sudo apt install apache2 -y
    sudo apt install openssh-server -y
    sudo systemctl enable apache2
    sudo systemctl enable ssh
    sudo systemctl start apache2
    sudo systemctl start ssh

Get IP address

    ip a | grep "192.168"

## Phase 3 — Kali Linux Attacker VM

Download Kali UTM image for Apple Silicon from
https://www.kali.org/get-kali/#kali-virtual-machines

UTM Settings
- Import the .utm file directly
- Network: Shared Network
- Default credentials: kali/kali

Update Kali and extract wordlist

    sudo apt update && sudo apt upgrade -y
    sudo gunzip /usr/share/wordlists/rockyou.txt.gz

## Phase 4 — Network Verification

Both VMs must be on Shared Network in UTM.
Test connectivity from Kali:

    ping -c 3 192.168.64.21

## Phase 5 — Universal Forwarder on Ubuntu

Download

    wget -O splunkforwarder.tgz "https://download.splunk.com/products/universalforwarder/releases/10.2.3/linux/splunkforwarder-10.2.3-4d61cf8a5c0c-linux-arm64.tgz"

Install and start

    sudo tar -xvzf splunkforwarder.tgz -C /opt
    sudo /opt/splunkforwarder/bin/splunk start --accept-license

Point to Splunk on Mac

    sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.64.1:9997 -auth admin:yourpassword

Add log sources

    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/access.log
    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/error.log

Restart and enable auto-start

    sudo /opt/splunkforwarder/bin/splunk restart
    sudo /opt/splunkforwarder/bin/splunk enable boot-start

## Phase 6 — Test Full Pipeline

Run attack from Kali

    hydra -l rishi -P /usr/share/wordlists/rockyou.txt ssh://192.168.64.21 -t 4 -V

Detect in Splunk on Mac at http://localhost:8000

    index=* "Failed password" | table _time, host, source

If events appear within 30 seconds the full pipeline is working.
