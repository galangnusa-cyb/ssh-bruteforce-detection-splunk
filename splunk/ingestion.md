# Splunk ingestion (Monitor auth.log)

## Splunk input settings
- **Path:** /var/log/auth.log
- **Sourcetype:** linux_secure
- **Index:** main
- **Host:** nikoserver

## Permission fix (if Splunk can’t read auth.log)
```bash
sudo usermod -aG adm splunk
sudo -u splunk /opt/splunk/bin/splunk restart
```

## Sanity check in Splunk Search
```bash
index=main host=nikoserver sourcetype=linux_secure "Failed password"
```

---

## Alert configuration (Scheduled)
- Type: Scheduled
- Schedule: Every 1 minute (cron */1 * * * *)
- Trigger: Number of results > 0
- Throttle: Suppress by src_ip for 5 minutes

## Dashboard (3 panels)
1. Failed SSH logins over time
2. Top source IPs by failures
3. Top targeted usernames

## Results
- Brute-force attempts from 192.168.68.129 were ingested and detected
- Detection rule flagged attacker IP when threshold exceeded
- Scheduled alert successfully triggered
- Dashboard showed spikes aligned with Hydra runs
