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
