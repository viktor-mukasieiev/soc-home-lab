# Phase 5 — Splunk Universal Forwarder

## Objective
Install and configure Splunk Universal Forwarder on Windows 11
to forward logs to Splunk Enterprise on Ubuntu.

## Software
- Splunk Universal Forwarder 9.4.0
- Installed on: Windows 11 VM (192.168.56.102)
- Forwarding to: 192.168.56.1:9997

## Configuration Files

### outputs.conf
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.56.1:9997

### inputs.conf
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = main
sourcetype = WinEventLog:Sysmon
renderXml = false
disabled = false

[WinEventLog://Security]
index = main
sourcetype = WinEventLog:Security
disabled = false

[WinEventLog://System]
index = main
sourcetype = WinEventLog:System
disabled = false

## Issues and Fixes
- user-seed.conf doesn't work after initial install
- Sysmon channel permissions fixed via wevtutil
- SplunkForwarder changed to run as LocalSystem

## Final Results
- 23,067 events indexed
- WinEventLog:Security  → 11,531 events
- WinEventLog:Sysmon   → 10,996 events
- WinEventLog:System   → 548 events

## Next Steps
- Run first Kali attack simulation
- Detect attack in Splunk
