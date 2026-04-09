# Case 12 — BlueSky Ransomware

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics  
**Difficulty:** Medium (Retired)  
**Evidence:** Network capture (`BlueSkyRansomware.pcap` — 2.10 MB) + Windows Event Log (`BlueSkyRansomware.evtx` — 1.06 MB)  
**Case Number:** 2026-012  

---

## Scenario

A suspected ransomware incident. Network traffic and event logs were captured. Reconstruct what happened — how the attacker got in, what they did before deploying ransomware, and what was left behind.

---

## What I Found

This was the most complete attack chain I'd investigated — reconnaissance, initial access, privilege escalation, defence evasion, credential dumping, persistence, lateral movement preparation, and finally ransomware deployment. Every phase present, all of it documented.

The PCAP started with a port scan. Then MSSQL authentication using the `sa` account — the default system administrator account that should never be exposed to external access. Once in, the attacker ran a SQL query to enable `xp_cmdshell`, which flips the MSSQL service from "database tool" to "OS command executor." From that point the database is a remote shell.

The event log was where I found the most significant piece. **Event ID 400** — PowerShell Engine Lifecycle — showed PowerShell being initialised inside `winlogon.exe`, with `HostName=MSFConsole`. That field is the giveaway: MSFConsole is the Metasploit Framework console. The attacker was running Metasploit and had managed to inject PowerShell into the Windows logon process. The timestamp was `2024-04-23 15:31:18` — **five days before the PCAP activity**.

That gap changed how I read the entire investigation. The PCAP wasn't capturing the beginning of the attack. The attacker had already been on the system for five days. The PCAP was capturing the deployment phase.

Back in the network traffic, the PowerShell payloads came in as Base64-encoded downloads. `checking.ps1` decoded to commands disabling Windows Defender. `del.ps1` decoded to a scheduled task creation — `\Microsoft\Windows\MUI\LPupdate`, disguised as a Windows language pack update task. That name is chosen specifically to look routine in a task list. The task kept the attacker's foothold alive across reboots.

`Invoke-PowerDump.ps1` was the credential dumping script. Hashes saved to `hashes.txt`. Then `extracted_hosts.txt` — a compiled list of internal network targets. The attacker was preparing for lateral movement.

The final download was `javaw.exe` — the ransomware binary. VirusTotal confirmed it as **BlueSky ransomware**, and sandbox analysis identified the ransom note: `# DECRYPT FILES BLUESKY #.txt`.

---

## Artifacts Documented

| # | Artifact | Source | Key Finding |
|---|----------|--------|-------------|
| 1 | Port scan | PCAP | `87.96.21.84` scanning `87.96.21.81` — reconnaissance |
| 2 | MSSQL login | PCAP | `sa` account authenticated — initial access via database service |
| 3 | xp_cmdshell | PCAP | OS command execution enabled through MSSQL |
| 4 | PowerShell in winlogon.exe | Event Log (Event ID 400) | `HostName=MSFConsole` — Metasploit-driven SYSTEM execution 5 days prior |
| 5 | checking.ps1 | PCAP (HTTP GET) | First payload — Windows Defender disabled |
| 6 | Defence evasion | Decoded payload | Defender disabled before further payloads delivered |
| 7 | del.ps1 / Persistence | Decoded payload | Scheduled task `\Microsoft\Windows\MUI\LPupdate` created |
| 8 | Invoke-PowerDump.ps1 | PCAP (HTTP GET) | Credential dumping script retrieved |
| 9 | hashes.txt | Decoded payload | NTLM credential hashes extracted and stored |
| 10 | extracted_hosts.txt | PCAP / script | Internal host list compiled — lateral movement preparation |
| 11 | javaw.exe | PCAP (HTTP GET) | BlueSky ransomware binary downloaded |
| 12 | Ransom note | VirusTotal sandbox | `# DECRYPT FILES BLUESKY #.txt` — encryption confirmed |
| 13 | Malware attribution | VirusTotal hash | javaw.exe = BlueSky ransomware family |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Wireshark 4.6.4 | PCAP analysis — TDS/MSSQL, HTTP streams, payload extraction |
| CyberChef 10.23.0 | Base64 decoding of obfuscated PowerShell payloads |
| Event Viewer | Windows Event Log analysis — PowerShell execution context |
| VirusTotal 2.5.8 | Malware attribution and sandbox behavioural analysis |

---

## What I Learned

**The event log revealed something the PCAP couldn't.** Process injection into `winlogon.exe` doesn't generate network traffic. The only evidence was in the host-based log — Event ID 400, `HostApplication=winlogon.exe`, `HostName=MSFConsole`. Without the EVTX file, I would never have known about the five-day prior access. This was the clearest demonstration I've had yet that network capture alone is not enough.

**The five-day gap is the most important finding in the report.** It tells you the PCAP wasn't capturing the beginning of the attack. The attacker had SYSTEM-level Metasploit access nearly a week before ransomware was deployed — time to do anything: exfiltrate data, map the network, set up persistence, dump credentials. In a real incident response, that gap would immediately expand the scope of the investigation.

**Base64 decoding is not optional.** Both `checking.ps1` and `del.ps1` looked like unremarkable HTTP downloads from the PCAP. Their actual content — Defender disabling and persistence creation respectively — only became visible after decoding. The filename tells you nothing. The content tells you everything.

**Scheduled task names are chosen to deceive.** `\Microsoft\Windows\MUI\LPupdate` sits in a path that looks like a legitimate Windows update mechanism. If you're scanning scheduled tasks quickly, you glance at it and move on. The right habit is to always verify what the task actually executes, not just what it's named.

**The PCAP captured the deployment phase, not the intrusion.** That's a real-world constraint you have to account for when scoping an investigation. If an attacker has been in your environment for five days and you only have network captures from day six, your forensic picture is partial. Host-based forensics — memory, event logs, registry — fills in what network visibility misses.

---

[View Full Report](./BlueSky-Ransomware-Report.pdf)
