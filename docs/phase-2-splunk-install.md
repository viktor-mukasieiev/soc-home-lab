# Phase 2 — Splunk Installation and Configuration

## Objective
Install and configure Splunk Enterprise 10.4.0 on Ubuntu host machine.

## Environment
- Host OS: Ubuntu Linux
- Splunk Version: 10.4.0
- Listening Port: 9997

## Commands Used
sudo dpkg -i splunk-10.4.0-f798d4d49089-linux-amd64.deb
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root
sudo /opt/splunk/bin/splunk enable boot-start --run-as-root
sudo ss -tlnp | grep 9997

## Results
- Splunk web UI accessible at http://localhost:8000
- Port 9997 confirmed listening (verified with ss -tlnp)
- Boot-start enabled successfully

## VM Network Configuration (Windows 11)
- Adapter 1: Host-Only (vboxnet0) — 192.168.56.0/24
- Adapter 2: Internal Network (SOC-Lab)

## Known Issues
- Splunk running as root (acceptable for home lab)
- Principle of Least Privilege violation — fix in later phase

## Next Steps
- Install Kali Linux VM
- Install Sysmon on Windows 11 VM
- Configure Splunk Universal Forwarder
