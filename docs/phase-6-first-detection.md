# Phase 6 — First Attack Detection

## Date
2026-07-20

## Attack Simulated
SMB Enumeration from Kali Linux

## MITRE ATT&CK
- T1046 — Network Service Discovery (Nmap)
- T1021.002 — SMB/Windows Admin Shares (smbclient)

## Attack Commands Used
# Reconnaissance
nmap -sV 192.168.56.102

# SMB Enumeration  
smbclient -L //192.168.56.102 -N

## Nmap Results
PORT     STATE  SERVICE
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds (SMB)

## Detection in Splunk
Search used:
index=main sourcetype="WinEventLog:Security"
(EventCode=4625 OR EventCode=4624 OR EventCode=4776)
earliest=-10m

## Evidence Found
- EventCode: 4625 (Failed Logon)
- Account_Name: kali
- Workstation_Name: KALI-ATTACKER
- Time: 2026-07-20 15:28:44

## Verdict
✅ ATTACK DETECTED SUCCESSFULLY

## What a Real Attacker Would Do Next
- Brute force SMB credentials
- Exploit EternalBlue (MS17-010) on port 445
- Attempt lateral movement

## What a Real SOC Analyst Would Do Next
- Create alert for EventCode 4625 > 5 times in 1 minute
- Block source IP 192.168.56.101 at firewall
- Open incident ticket
- Escalate to Tier 2 analyst
