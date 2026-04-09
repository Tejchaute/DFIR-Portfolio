# Case 11 — Lockdown

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics  
**Difficulty:** Easy (Retired)  
**Evidence:** Network capture (`capture.pcapng` — 3.7 MB) + Memory dump (`memdump.mem` — 4.5 GB) + Malware sample (`updatenow.exe` — 588 KB)  
**Case Number:** 2026-011  

---

## Scenario

An IIS web server was potentially compromised. Three pieces of evidence were provided: a network capture, a memory dump, and a recovered executable. Figure out how the attacker got in, what they did, and what they left behind.

---

## What I Found

Three evidence sources, each answering different questions. This was the most complex lab I'd done at the point I completed it — not because any individual step was hard, but because you had to hold three separate investigations in your head and work out how they connected.

The PCAP told the entry story. The attacker at `10.0.2.4` started with a TCP SYN scan against the IIS server at `10.0.2.15`, then moved to SMB. They accessed `\\10.0.2.15\IPC$` and then `\\10.0.2.15\Documents`, enumerating what was writable. Then at `05:48:52` they uploaded `shell.aspx` via SMB. Not through a web vulnerability — they just wrote directly to a location IIS would serve. That distinction matters: the attack path here was misconfigured SMB access, not an application flaw.

The reverse shell came from that web shell. At `06:08:23` the server initiated an outbound connection to the attacker on port 4443 — same direction-reversal trick as WebStrike, bypassing inbound firewall rules. The web shell had given them enough to execute commands, and those commands set up persistent control.

The memory dump filled in what happened after the shell was established. Volatility 3's `pslist` showed `w3wp.exe` — the IIS worker process — had spawned `updatenow.exe`. An IIS worker process spawning an arbitrary executable is one of those process-parent relationships that should never appear in a normal environment. That's the web shell executing the second-stage payload.

The executable itself was `updatenow.exe` — the standalone malware sample in the evidence. `strings` on it immediately showed `UPX` in the header: packed binary. VirusTotal with the SHA256 hash came back clean and fast: **Agent Tesla**. A credential-stealing RAT that targets browsers, email clients, and FTP applications through keylogging and screenshot capture.

Persistence came from the Startup folder: `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe`. World-writable in many configurations, survives reboots, no admin rights needed.

The hardest part of this investigation was C2 attribution. DNS analysis and memory string extraction both produced too much noise to pinpoint the C2 domain reliably. VirusTotal's behavioural analysis of the hash gave me what I needed — a reminder that local evidence isn't always sufficient and threat intel platforms exist for exactly this gap.

---

## Artifacts Documented

| # | Artifact | Source | Key Finding |
|---|----------|--------|-------------|
| 1 | TCP SYN scan | PCAP | `10.0.2.4` scanning `10.0.2.15` — reconnaissance begins |
| 2 | SMB IPC$ access | PCAP | Authentication and admin share access confirmed |
| 3 | SMB Documents access | PCAP | Internal share enumeration — identifying writable locations |
| 4 | Web shell upload | PCAP | `shell.aspx` written to IIS server via SMB |
| 5 | Reverse shell | PCAP | Outbound TCP to attacker port 4443 — C2 established |
| 6 | Process execution | Memory | `w3wp.exe` → `updatenow.exe` — web shell executing payload |
| 7 | Persistence | Memory/filesystem | `updatenow.exe` in Startup folder — survives reboot |
| 8 | UPX packing | strings analysis | Packed binary — obfuscation confirmed |
| 9 | Malware attribution | VirusTotal | Agent Tesla — credential-stealing RAT |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Wireshark 4.6.4 | PCAP analysis — TCP streams, SMB traffic, C2 identification |
| Volatility 3 (2.28.1) | Memory analysis — process tree, persistence detection |
| strings (2.46) | Static string extraction from updatenow.exe |
| VirusTotal (2.5.8) | Hash-based malware attribution |

---

## What I Learned

**The attack didn't need a vulnerability.** The attacker wrote directly to a writable SMB share that happened to be in IIS's serving path. No SQL injection, no XSS, no CVE. Just overly permissive file sharing. That's a more common attack path in real environments than most people expect, and it's a reminder that network-level access controls are just as important as application security.

**w3wp.exe spawning anything unusual is a high-signal detection rule.** IIS worker processes manage web requests. They don't spawn executables. The moment `w3wp.exe` appears as a parent of something that isn't an IIS component, you have evidence of web shell post-exploitation. That's a detection rule worth remembering and one you can implement in almost any EDR.

**UPX packing shows up immediately in strings output.** The string 'UPX' appears in the binary header. You don't need a specialised unpacker to identify it — basic string extraction catches it. The packed binary itself is harder to analyse statically, but VirusTotal's sandbox handles the unpacking automatically, which is why hash submission was faster than trying to do static analysis on the packed version locally.

**When local analysis produces noise, pivot to threat intel.** DNS analysis and memory strings for C2 attribution gave me too many false positives to be useful. The SHA256 hash submitted to VirusTotal gave me Agent Tesla in seconds. Knowing when to stop extracting locally and start using external intelligence is a practical skill, not a shortcut.

---

[View Full Report](./Lockdown-Report.pdf)
