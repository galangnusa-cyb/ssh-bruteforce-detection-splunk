OVERVIEW
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



=== SETUP ===

= Enable SSH on Ubuntu:

sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh

= Install Hydra:

sudo apt update
sudo apt install -y hydra

= Create small password list:

printf "123456\npassword\nqwerty\nadmin\nletmein\nkali\n" > pass.txt

= Run brute-force:

hydra -l <ubuntu_user> -P pass.txt -t 8 -V ssh://192.168.68.128

= Verify logs on Ubuntu:

sudo grep "Failed password" /var/log/auth.log | tail -n 30



=== SPLUNK INGESTION ===

Monitored file:
- Path: /var/log/auth.log
- Sourcetype: linux_secure
- Index: main
- If Splunk can’t read the file:

sudo usermod -aG adm splunk
sudo -u splunk /opt/splunk/bin/splunk restart

=== DETECTION (SPL) ===

= Brute-force Threshold: ≥10 failures in 5 minutes:

index=main host=nikoserver sourcetype=linux_secure "Failed password" "sshd"
| rex "Failed password for (invalid user )?(?<user>\S+) from (?<src_ip>\d{1,3}(\.\d{1,3}){3})"
| bin _time span=5m
| stats count as fail_count dc(user) as uniq_users values(user) as users by src_ip _time host
| where fail_count >= 10
| sort - fail_count

(brute-force behavior typically produces many failed logins from a single source IP in a short window)

Alert Configuration:
- Type: Scheduled
- Runs: Every 1 minute (cron */1 * * * *)
- Trigger: Number of results > 0
- Throttle: suppress by src_ip for 5 minutes

Dashboard Panels:
- Failed SSH logins over time
- Top source IPs by failed logins
- Top targeted usernames

Result:
- Successfully detected brute-force attempts originating from attacker IP (192.168.68.129)
- Triggered alert when threshold exceeded
- Dashboard shows clear spikes correlated with Hydra runs

Improvements (Next Steps)
- Add correlation: “success after many failures”
- Add allowlist for trusted IPs to reduce false positives
- Enrich attacker IP (geo/whois) and add risk scoring
- Forward detections into a dedicated alerts index
