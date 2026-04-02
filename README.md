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

*New cases added as I progress through the learning path.*

---

## Skills Built So Far

**Disk Forensics**
- Windows artifact analysis — Registry hives (SAM, SYSTEM, NTUSER.DAT), Prefetch files, LNK files, UserAssist, Jump Lists, Shellbags, Recycle Bin
- Windows credential forensics — NTLM hash extraction via Mimikatz, offline cracking with Hashcat
- Linux forensics — bash history, auth logs, dpkg logs, cron jobs, filesystem artifact recovery
- Image acquisition and analysis with FTK Imager and Autopsy
- External device forensics — USBSTOR registry keys, EXIF metadata, device identification without physical access

**Memory Forensics**
- Volatility 2 on Linux memory dumps — process trees, network connections, bash history, syscall table inspection, kernel module analysis
- Identifying live C2 connections, reverse shells, and rootkits from RAM
- Recovering artifacts that don't exist on disk

**Cross-Source Investigation**
- Correlating disk and memory evidence from the same machine to reconstruct a complete attack chain
- Identifying which questions each source can and cannot answer independently
- Building unified timelines from filesystem timestamps and memory artifacts

**Investigation and Reporting**
- Writing formal DFIR reports — artifact tables, event timelines, chain of custody
- Reconstructing attack chains from raw evidence across multiple artifact sources
- Documenting methodology to professional standards

---

## Toolset

| Category | Tools |
|----------|-------|
| Disk analysis | Autopsy, FTK Imager, Sleuth Kit |
| Memory analysis | Volatility 2, strings, grep |
| Registry parsing | RegRipper, Registry Explorer, Mimikatz |
| Hash cracking | Hashcat |
| File carving | Foremost, PhotoRec |
| Browser forensics | DB Browser for SQLite, BrowsingHistoryView, Hindsight |
| Metadata analysis | EXIF Viewer |
| Log parsing | EvtxECmd, JLECmd, WinPrefetchViewer |
| Network forensics | Wireshark, NetworkMiner *(learning)* |
| Environment | SIFT Workstation (Ubuntu) |

---

## Learning Path

```
Phase 1 — Foundations     [Completed]    OS internals, networking, Linux, Python basics
Phase 2 — Core Forensics  [In Progress]  Disk, memory, network, mobile forensics

```

---

## Connect

- LinkedIn: [linkedin.com/in/tej-chaute](https://www.linkedin.com/in/tej-chaute)
- Reports are attached as PDFs inside each case folder
