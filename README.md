# DFIR Portfolio — Tej Chaute
> Aspiring Digital Forensics Analyst | Building hands-on investigation skills through structured lab work and real-world style case analysis.

---

## About

This repository documents my journey through digital forensics and incident response. Every case here is a real investigation I worked through — not just challenge completions, but full reports with artifact tables, event timelines, and documented methodology.

I started this in early 2026 as part of a structured learning path toward becoming a DFIR analyst. Each case folder has a write-up of what I found, what tools I used, and what I actually learned from it — written the way I'd explain it to someone, not the way a textbook would.


---

## Cases

| # | Case | Evidence Type | Platform | Difficulty | Key Skills |
|---|------|--------------|----------|------------|------------|
| 01 | [Hunter](./01-Hunter/) | Windows disk image | CyberDefenders | Medium | Registry, Prefetch, Skype DB, anti-forensics |
| 02 | [Insider](./02-Insider/) | Linux disk image | CyberDefenders | Easy | Bash history, auth logs, FTK Imager |
| 03 | [Seized](./03-Seized/) | Linux memory dump | CyberDefenders | Medium | Volatility 2, rootkit detection, SSH persistence |
| 04 | [Ulysses](./04-Ulysses/) | Linux disk + memory dump | CyberDefenders | Medium | Disk + memory correlation, Exim4 exploitation, netcat backdoor |
| 05 | [AfricanFalls](./05-AfricanFalls/) | Windows disk image | CyberDefenders | Medium | Browser history, NTLM hash cracking, EXIF metadata, USB forensics |
| 06 | [Phishy](./06-Phishy/) | Windows disk image | CyberDefenders | Medium | Phishing analysis, VBA macros, WhatsApp forensics, credential harvesting |
| 07 | [Sysinternals](./07-Sysinternals/) | Windows disk image (E01) | CyberDefenders | Medium | AmCache, ShimCache, malware execution tracing, service persistence |
| 08 | [WebStrike](./08-WebStrike/) | Network capture (PCAP) | CyberDefenders | Easy | Wireshark, web shell analysis, reverse shell, data exfiltration |
| 09 | [PacketMaze](./09-PacketMaze/) | Network capture (PCAP) | CyberDefenders | Medium | FTP forensics, TLS analysis, EXIF metadata, multi-protocol investigation |
| 10 | [Reveal](./10-Reveal/) | Windows memory dump | CyberDefenders | Easy | Volatility 3, fileless attack, LOLBins, reflective DLL injection |
| 11 | [Lockdown](./11-Lockdown/) | PCAP + memory dump + malware sample | CyberDefenders | Easy | Multi-source correlation, web shell, Agent Tesla, UPX analysis |
| 12 | [BlueSky](./12-BlueSky/) | PCAP + Windows Event Log | CyberDefenders | Medium | MSSQL exploitation, ransomware lifecycle, event log forensics |

*New cases added as I progress through the learning path.*

---

## Skills Built So Far

**Disk Forensics**
- Windows artifact analysis — Registry hives (SAM, SYSTEM, NTUSER.DAT), Prefetch files, LNK files, UserAssist, Jump Lists, Shellbags, Recycle Bin
- Windows credential forensics — NTLM hash extraction via Mimikatz, offline cracking with Hashcat, saved browser credential recovery with PasswordFox
- Windows execution tracing — ShimCache (AppCompatCache), AmCache, Prefetch cross-validation; understanding what each records and what each doesn't
- Linux forensics — bash history, auth logs, dpkg logs, cron jobs, filesystem artifact recovery
- Image acquisition and analysis with FTK Imager and Autopsy; direct E01 mounting in Autopsy
- External device forensics — USBSTOR registry keys, EXIF metadata, device identification without physical access
- Phishing attack reconstruction — malicious document analysis, VBA macro deobfuscation, multi-stage payload tracing
- Malware analysis from disk — trojanised executables, dropper behaviour, UPX unpacking, service-based persistence, masquerading

**Memory Forensics**
- Volatility 2 on Linux memory dumps — process trees, network connections, bash history, syscall table inspection, kernel module analysis
- Volatility 3 on Windows memory dumps — pslist, psscan, cmdline, getsids, netscan, malfind
- Identifying live C2 connections, reverse shells, and rootkits from RAM
- Fileless attack detection — LOLBin abuse (PowerShell, rundll32, net.exe), WebDAV payload delivery, reflective DLL loading
- Recovering artifacts that don't exist on disk

**Network Forensics**
- PCAP analysis with Wireshark — HTTP stream reconstruction, TCP stream following, FTP session recovery, TDS/MSSQL inspection
- Web application attack analysis — file upload exploitation, double extension bypass, web shell deployment
- Reverse shell identification — recognising outbound C2 connections and netcat payloads in traffic
- Data exfiltration detection — curl-based POST exfiltration, FTP file retrieval, cleartext credential exposure
- FTP forensics — passive mode (PASV) data channel recovery, credential extraction, file transfer reconstruction
- TLS session analysis — SNI-based destination identification, client random extraction for potential decryption
- Attacker attribution — IP geolocation, connection direction analysis, MAC address vendor lookup

**Event Log Forensics**
- Windows Event Log analysis with Event Viewer — PowerShell Engine Lifecycle (Event ID 400), process execution context
- Correlating event log timestamps with PCAP activity to identify multi-phase attacks
- Identifying process injection and SYSTEM-context execution from host-based log artifacts

**Malware Analysis**
- Static analysis — string extraction, UPX packing identification, hash-based threat intelligence
- Behavioural attribution via VirusTotal sandbox — ransomware family identification, C2 infrastructure mapping
- Multi-stage payload tracing — dropper → downloader → ransomware chains

**Cross-Source Investigation**
- Correlating disk, memory, network, and event log evidence to reconstruct complete attack chains
- Identifying which questions each source can and cannot answer independently
- Building unified timelines across filesystem timestamps, memory artifacts, packet captures, and event logs

**Investigation and Reporting**
- Writing formal DFIR reports — artifact tables, event timelines, chain of custody
- Reconstructing attack chains from raw evidence across multiple artifact sources
- Documenting methodology, tool versions, and analytical limitations to professional standards

---

## Toolset

| Category | Tools |
|----------|-------|
| Disk analysis | Autopsy, FTK Imager, Sleuth Kit |
| Memory analysis | Volatility 2, Volatility 3, strings, grep |
| Registry parsing | RegRipper, Registry Explorer, Mimikatz |
| Execution analysis | AppCompatCacheParser, AmcacheParser, WinPrefetchViewer |
| Hash cracking | Hashcat |
| File carving | Foremost, PhotoRec |
| Browser forensics | DB Browser for SQLite, BrowsingHistoryView, Hindsight, PasswordFox |
| Document analysis | oledump, olevba, CyberChef |
| Messaging forensics | WhatsApp Viewer |
| Metadata analysis | EXIF Viewer |
| Threat intelligence | VirusTotal |
| Network forensics | Wireshark, NetworkMiner |
| Log parsing | EvtxECmd, JLECmd, Event Viewer |

---

## Learning Path

```
Phase 1 — Foundations     [Completed]    OS internals, networking, Linux, Python basics
Phase 2 — Core Forensics  [In Progress]  Disk, memory, network, malware, event log forensics

```

---

## Connect

- LinkedIn: [linkedin.com/in/tej-chaute](https://www.linkedin.com/in/tej-chaute)
- Reports are attached as PDFs inside each case folder
