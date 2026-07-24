# Phase 8 — Attack Simulations

## Date
2026-07-24

## Attack 1 — Nmap Aggressive Reconnaissance
MITRE ATT&CK: T1046, T1082

Command:
nmap -A -sV -sC -O 192.168.56.102

Intelligence gathered:
- OS: Windows 11 21H2 (94% confidence)
- Computer name: WINTARGET
- Open ports: 135, 139, 445, 3389
- RDP certificate: commonName=wintarget
- SMB signing: enabled and required
- Product version: 10.0.26100

Detection in Splunk:
- Sysmon EventCode 3 (Network Connection)
- 10 events from 192.168.56.101 → port 3389
- Search: index=main sourcetype="WinEventLog:Sysmon" EventCode=3

## Attack 2 — RDP Brute Force
MITRE ATT&CK: T1110.003

Command:
hydra -l administrator -P /usr/share/wordlists/rockyou.txt 
rdp://192.168.56.102 -t 1 -V -f

Result:
- Windows rate-limited and blocked connections
- 23 EventCode 4625 events in 5 minutes
- Account targeted: administrator
- Source: kali-attacker

Detection in Splunk:
index=main sourcetype="WinEventLog:Security" EventCode=4625 
earliest=-5m | table _time, Account_Name, Workstation_Name

## Complete Kill Chain Detected
T1046 → Reconnaissance → Detected ✅
T1082 → System Discovery → Detected ✅
T1110.001 → SMB Brute Force → Detected ✅
T1110.003 → RDP Brute Force → Detected ✅

## What Real SOC Analyst Does
1. Correlate Nmap scan with subsequent brute force
2. Identify attacker IP: 192.168.56.101
3. Block source IP at firewall immediately
4. Reset administrator password
5. Enable account lockout policy
6. Open P1 incident ticket
7. Check for successful logons from same IP

## Attack 3 — Metasploit MS17-010 (EternalBlue) Scan
MITRE ATT&CK: T1210, T1555.004

Command:
msfconsole -q -x "use auxiliary/scanner/smb/smb_ms17_010; 
set RHOSTS 192.168.56.102; run; exit"

Result:
- Windows 11 is patched — NOT vulnerable to MS17-010 ✅
- SMB Login Error on port 445

New EventCode Detected: 5379
- Credential Manager credentials were read
- 14 events in rapid succession
- Accounts: WINTARGET$ and vboxuser
- MITRE ATT&CK T1555.004 — Windows Credential Manager

## Full Detection Summary — Phase 8
T1046 → Nmap Recon → Sysmon EventCode 3 ✅
T1110 → SMB Brute Force → EventCode 4625 (26 events) ✅
T1110.003 → RDP Brute Force → EventCode 4625 (23 events) ✅
T1555.004 → Credential Access → EventCode 5379 (14 events) ✅
