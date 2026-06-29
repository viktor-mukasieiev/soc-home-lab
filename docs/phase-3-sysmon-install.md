# Phase 3 — Sysmon Installation on Windows 11

## Objective
Install Sysmon with SwiftOnSecurity config for rich 
Windows telemetry.

## Software Versions
- Sysmon: v15.21
- Config: SwiftOnSecurity sysmonconfig-export.xml
- Schema: 4.91

## Installation Command
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml

## Verification
Get-Service Sysmon64 → Status: Running
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" 
-MaxEvents 10 → Events confirmed

## Events Confirmed Working
- Event ID 1  → Process Create
- Event ID 5  → Process Terminated  
- Event ID 11 → File Created
- Event ID 13 → Registry Modified
- Event ID 22 → DNS Query

## Next Step
Install Splunk Universal Forwarder on Windows 11
