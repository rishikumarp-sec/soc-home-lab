# SPL Detection Queries

## 1. Brute Force Detection
Counts failed SSH login attempts grouped by source IP.

    index=* "Failed password" | stats count by src_ip | sort -count

## 2. Attack Timeline
Shows volume of attacks over time as a chart.

    index=* "Failed password" | timechart count

## 3. Extract Attacker IP
Pulls the attacking IP address out of raw log data.

    index=* "Failed password"
    | rex "from (?P<src_ip>\d+\.\d+\.\d+\.\d+)"
    | stats count by src_ip

## 4. Web Attack Detection
Finds suspicious web requests in Apache logs.

    index=* source="*apache*" | table _time, host, _raw

## 5. Privilege Escalation Detection
Finds failed sudo attempts on the victim machine.

    index=* "sudo" "incorrect password" | table _time, host, _raw

## 6. High Volume Source IPs
Finds any IP making unusually high number of requests.

    index=* | stats count by src_ip | where count > 50 | sort -count
