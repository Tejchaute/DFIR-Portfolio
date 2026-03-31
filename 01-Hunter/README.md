# Case 01 — Hunter

**Platform:** CyberDefenders  
**Difficulty:** Easy  
**Evidence:** Windows disk image (Hunter.ad1 — 606 MB)  
**Case Number:** 2026-001  
**Date:** 20/03/2026  

---

## What was the case about?

The SOC flagged illegal port scanning coming from an employee's machine. My job was to figure out if it was intentional, what tools they used, and whether there was anything else going on.

Spoiler: there was a lot more going on.

---

## What I found

The port scan was just the tip of it. Yes, the user ran Nmap v7.12 through Zenmap and scanned 1000 ports — the results were literally saved as an XML file on the Desktop. But digging further, the whole thing turned into a full insider threat investigation.

The user had been talking to someone on Skype (account: `linux-rul3z`) and they explicitly agreed to use TeamViewer for remote access and data exfiltration. I found this in the Skype SQLite database — the messages were still sitting there in `main.db`. The account was linked to `ehptmsgs@gmail.com`.

Beyond that there was a dedicated folder at `C:\Users\Hunter\Pictures\Exfil` for staging data, a compressed archive (`Pictures.7z`) sent as an email attachment, Tor Browser executed to stay anonymous, and Burp Suite downloaded for traffic interception.

The anti-forensics angle was the most interesting part. Jetico BCWipe (a file shredder) had been installed almost a month before the incident — 2016-05-23 — and was run 5 times on the day itself. Crypto Swap (disk encryption) was also installed that same day. The pre-installation date is what made this clearly premeditated rather than impulsive.

---

## Artifacts that mattered most

| Artifact | Path | What it showed |
|----------|------|----------------|
| Prefetch files | `C:\Windows\Prefetch\` | ZENMAP.EXE and BCWIPE.EXE — run counts and timestamps |
| Nmap XML output | `C:\Users\hunter\Desktop\nmapscan.xml` | Full scan results, tool version, open ports (22, 80, 9929, 31337) |
| Skype database | `C:\Users\hunter\AppData\Roaming\Skype\hunterehpt\main.db` | Exfiltration agreement with external party, linked email |
| Registry USBSTOR | `SYSTEM\ControlSet001\Enum\USBSTOR` | Two USB devices connected — serial numbers preserved |
| Jump Lists | `AppData\...\CustomDestinations\` | Tor Browser execution confirmed |
| Shellbags | NTUSER.DAT | Exfil staging folder was browsed |
| Recycle Bin | `C:\$Recycle.Bin\` | Deleted files with original paths and timestamps |
| Event Logs | `Security.evtx` (Event ID 4624) | Login history and last login time |

---

## Tools used

| Tool | Version |
|------|---------|
| Autopsy | 4.22.1 |
| FTK Imager | 4.7.3.81 |
| Registry Explorer | 2.1.0.0 |
| EvtxECmd | 1.5.2.0 |
| WinPrefetchViewer | 1.3.7 |
| DB Browser for SQLite | 3.13.1.0 |
| JLECmd | 1.5.1.0 |

---

## What I took away from this

The initial SOC alert was just port scanning. If I had stopped there I would have missed the Skype conversation, the staging folder, the exfiltration attempt, and the anti-forensic tools. The lesson I actually internalized here: the first artifact you find is rarely the whole story.

The Skype database was the most valuable find — not because it was technically difficult to parse, but because it gave me intent rather than just activity. Proving someone ran a tool is one thing. Proving they discussed using it for exfiltration with an external party is a completely different level of evidence.

---

[View Full Report](./Hunter-Report.pdf)
