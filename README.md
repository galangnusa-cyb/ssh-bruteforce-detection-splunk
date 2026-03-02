# SSH Brute-Force Detection in Splunk (Home Lab)

This project simulates SSH brute-force activity in an isolated VMware lab and builds Splunk detection + alerting using Linux authentication logs (`/var/log/auth.log`).

## What this project demonstrates
- Generating brute-force SSH telemetry (Kali → Ubuntu) using Hydra
- Ingesting `auth.log` into Splunk (sourcetype `linux_secure`)
- Detecting brute-force attempts with a threshold rule (**≥10 failures in 5 minutes**)
- Visualizing trends in a 3-panel Splunk dashboard
- Triggering a scheduled alert with throttling (per attacker IP)

## Lab architecture
- **Attacker:** Kali Linux (Hydra)
- **Target + Splunk:** Ubuntu Server (OpenSSH, `/var/log/auth.log`, Splunk Enterprise)
- **Networking:**
  - Host-only (lab): `192.168.68.0/24`
  - NAT (Splunk Web access): `192.168.28.0/24`

### IPs in this lab
- Ubuntu (Host-only): `192.168.68.128`
- Kali (Host-only): `192.168.68.129`
- Ubuntu (NAT / Splunk Web): `192.168.28.128` → `http://192.168.28.128:8000`

!All testing is performed in a local isolated lab!

---

## 1) Target setup (Ubuntu SSH)
```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

## 2) Verify
```bash
sudo systemctl status ssh --no-pager
sudo tail -n 20 /var/log/auth.log
```

## 3) Install Hydra
```bash
sudo apt update
sudo apt install -y hydra
```

## 4) Create a small password list
```bash
printf "123456\password\qwerty\admin\letmein\kali\nikoganteng" > pass.txt
```

## 5) Run brute-force
```bash
hydra -l niko -P pass.txt -t 8 -V ssh://192.168.68.128
```

## 6) Confirm log activity on Ubuntu
```bash
sudo grep "Failed password" /var/log/auth.log | tail -n 30
```
