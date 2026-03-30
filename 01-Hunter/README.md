# Case 01 — Hunter

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics  
**Difficulty:** Medium 
**Evidence:** Windows disk image (Hunter.ad1 — 606 MB)  
**Case Number:** 2026-001  
**Date:** 20/03/2026  

---

## Scenario

The SOC received an alert about illegal port scanning activity originating from an employee's workstation. The IR team acquired a full forensic image of the suspect's machine. The objective was to determine whether the scanning was intentional, identify what tools were used, and uncover any additional malicious activity.

---

## What I Found

The investigation revealed a deliberate and coordinated insider threat. The user conducted network reconnaissance using Nmap v7.12 via Zenmap, scanning 1000 ports on a target and identifying four open ports (22, 80, 9929, 31337). Evidence of the scan was preserved in a saved XML output file on the Desktop.

Beyond the port scan, the investigation uncovered a significantly broader attack plan:

- **External coordination** — Skype conversation with account `linux-rul3z` confirmed an explicit agreement to use TeamViewer for remote access and data exfiltration
- **Data staging** — a dedicated folder (`C:\Users\Hunter\Pictures\Exfil`) was created to stage files before exfiltration
- **Exfiltration attempt** — a compressed archive (`Pictures.7z`) was sent as an email attachment
- **Anonymisation** — Tor Browser was executed to evade network monitoring
- **Anti-forensic tools** — Jetico BCWipe (file shredder) was executed 5 times and Crypto Swap (disk encryption) was installed, both used to destroy evidence
- **Offensive tooling** — Burp Suite v1.7.03 was downloaded and executed

The attacker's pre-planning was confirmed by BCWipe being installed nearly a month before the incident date (2016-05-23), indicating this was not impulsive behaviour.

---

## Artifacts Investigated

| Artifact           | Location                                                       | Key Finding                                                                  |
|--------------------|----------------------------------------------------------------|------------------------------------------------------------------------------|
| Prefetch files     | `C:\Windows\Prefetch\`                                         | ZENMAP.EXE and BCWIPE.EXE execution confirmed with timestamps and run counts |
| Nmap XML output    | `C:\Users\hunter\Desktop\nmapscan.xml`                         | Full scan results — ports, tool version, scan end time                       |
| Skype database     | `C:\Users\hunter\AppData\Roaming\Skype\hunterehpt\main.db`     | Exfiltration agreement with external party, linked email                     |
| Registry — USBSTOR | `SYSTEM\ControlSet001\Enum\USBSTOR`                            | Two USB devices connected (serial numbers recorded)                          |
| Jump Lists         | `AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations\` | Tor Browser execution confirmed                                              |
| Shellbags          | NTUSER.DAT                                                     | Exfil staging folder browsed                                                 |
| Recycle Bin        | `C:\$Recycle.Bin\`                                             | Deleted files with original paths and deletion timestamps                    |
| Windows Event Logs | `Security.evtx` (Event ID 4624)                                | Login history and last login time                                            |

---

## Tools Used

| Tool                  | Version  | Purpose                           |
|-----------------------|----------|-----------------------------------|
| Autopsy               | 4.22.1   | Primary disk analysis             |
| FTK Imager            | 4.7.3.81 | AD1 image mounting and extraction |
| Registry Explorer     | 2.1.0.0  | Offline registry hive analysis    |
| EvtxECmd              | 1.5.2.0  | Event log parsing                 |
| WinPrefetchViewer     | 1.3.7.0  | Prefetch execution history        |
| DB Browser for SQLite | 3.13.1   | Skype main.db analysis            |
| JLECmd                | 1.5.1    | Jump List parsing                 |

---

## Key Takeaway

This case demonstrated that a single SOC alert (port scanning) was the visible tip of a much larger insider threat operation. The most important lesson: **always follow the evidence chain beyond the initial alert**. The Skype database and Shellbags analysis — neither directly related to port scanning — uncovered the full exfiltration plan.

BCWipe being pre-installed a month before the incident is what elevated this from "suspicious behaviour" to "premeditated insider threat."

---

## Full Report

[View Full Report](./Hunter-Report)
