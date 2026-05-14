# Incident Report 001 — SSH Brute Force Attack

## Incident Summary
| Field | Detail |
|-------|--------|
| Date | 2026-05-14 |
| Time | 16:32 UTC |
| Severity | High |
| Type | SSH Brute Force Attack |
| Status | Detected |

## Timeline
| Time | Event |
|------|-------|
| 16:32:42 | Hydra brute force initiated from Kali 192.168.64.14 |
| 16:32:46 | First failed login detected in auth.log |
| 16:32:46 | Universal Forwarder ships log to Splunk |
| 16:32:47 | Splunk indexes the event |
| 16:33:00 | SOC analyst detects pattern via SPL query |
| 16:33:10 | Attacker IP identified as 192.168.64.14 |

## Attack Details
- Attacker IP: 192.168.64.14 (Kali Linux VM)
- Target IP: 192.168.64.21 (Ubuntu Server VM)
- Target service: SSH port 22
- Tool used: Hydra v9.6
- Wordlist: rockyou.txt with 14,344,399 passwords
- Attack rate: 4 concurrent threads
- Username attempted: rishi

## Detection Method
Detected using Splunk SPL query monitoring auth.log

    index=* "Failed password"
    | rex "from (?P<src_ip>\d+\.\d+\.\d+\.\d+)"
    | stats count by src_ip
    | sort -count

## Evidence
- Multiple failed SSH authentication attempts in /var/log/auth.log
- All attempts originating from single source IP 192.168.64.14
- Attack pattern consistent with automated brute force tool
- 49 attempts detected within 30 seconds

## Impact Assessment
- No successful login achieved
- No data exfiltrated
- No persistence established
- System integrity maintained

## Recommendations
1. Implement fail2ban to auto-block IPs after 5 failed attempts
2. Disable password authentication and use SSH keys only
3. Change SSH from default port 22 to a custom port
4. Set up Splunk alert when failed logins exceed 10 in 1 minute
5. Whitelist only known IPs for SSH access

## Lessons Learned
- Brute force attacks generate high volume of auth.log entries
- Splunk Universal Forwarder shipped logs in real time successfully
- SPL regex can extract attacker IP from raw log data
- Early detection possible within seconds of attack initiation
