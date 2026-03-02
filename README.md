Overview

This project simulates SSH brute-force activity in a controlled lab environment and builds Splunk detections, dashboards, and alerts to identify credential abuse patterns using Linux authentication logs (/var/log/auth.log).

Goals
- Generate realistic SSH brute-force logs from an attacker host
- Ingest auth.log into Splunk
- Detect brute-force behavior using threshold-based SPL
- Visualize patterns in a dashboard (failed logins trend, top attacker IPs, targeted usernames)
- Trigger an alert when brute-force threshold is exceeded

Lab Environment
- Attacker: Kali Linux (Hydra)
- Target: Ubuntu Server (OpenSSH + /var/log/auth.log)
- SIEM: Splunk Enterprise (installed on Ubuntu)

Network:
- Host-only (lab): 192.168.68.0/24
- NAT (Splunk Web access): 192.168.28.0/24

IPs used (example in this lab):
- Ubuntu target (Host-only): 192.168.68.128
- Kali attacker (Host-only): 192.168.68.129
- Ubuntu NAT (Splunk Web): 192.168.28.128 → http://192.168.28.128:8000

!All testing performed in a local isolated lab!
