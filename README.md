# DFIR Portfolio — Tej Chaute

> Aspiring Digital Forensics Analyst | Building hands-on investigation skills through structured lab work and real-world style case analysis.

---

## About This Repository

This portfolio documents my progression through digital forensics and incident response. Each case includes a full investigation report — artifact tables, event timelines, tools used, and documented commands — written to professional DFIR standards.

---

## Case Studies

| # | Case | Platform | Evidence Type | Difficulty | Key Techniques |
|---|------|----------|---------------|------------|----------------|
| 01 | [Hunter](./01-Hunter/) | CyberDefenders | Windows Disk Image | Medium | Registry analysis, Prefetch, UserAssist, browser artifacts, LNK files |
| 02 | [Insider](./02-Insider/) | CyberDefenders | Linux Disk Image | Easy | FTK Imager, bash history, Linux logs, credential access |
| 03 | [Seized](./03-Seized/) | CyberDefenders | Linux Memory Dump | Medium | Volatility 2, reverse shell detection, SSH key persistence, rootkit analysis |

*New cases added weekly as I progress through the learning path.*

---

## Skills Demonstrated

**Disk Forensics**
- Windows NTFS artifact analysis — Registry hives, Prefetch files, LNK files, UserAssist, Recycle Bin
- Linux EXT4 forensics — bash history, auth logs, dpkg logs, cron jobs
- FTK Imager and Autopsy for image acquisition and analysis

**Memory Forensics**
- Volatility 2 plugins — linux_pstree, linux_psaux, linux_netstat, linux_bash, linux_check_syscall, linux_lsmod, linux_dump_map
- Process analysis, network connection mapping, rootkit detection
- Base64 payload decoding and attacker command reconstruction

**Investigation & Reporting**
- Formal DFIR report writing — artifact tables, event timelines, chain of custody documentation
- Attack chain reconstruction from raw forensic evidence
- Cross-source artifact correlation

---

## Toolset

| Category          | Tools |
|-------------------|-------|
| Disk analysis     | Autopsy, FTK Imager, Sleuth Kit |
| Memory analysis   | Volatility 2, strings, grep |
| Registry parsing  | RegRipper, Registry Explorer |
| File carving      | Foremost, PhotoRec |
| Network forensics | Wireshark, NetworkMiner *(in progress)* |
| Scripting         | Python, Bash, sqlite3 |
| Environment       | SIFT Workstation (Ubuntu) |

---

## Learning Path

```
Phase 1 — Foundations        [Completed] OS internals, networking, Linux, Python
Phase 2 — Core Forensics     [In Progress] Disk, memory, network, mobile forensics
```

---

## Connect

- LinkedIn: [linkedin.com/in/tej-chaute](https://www.linkedin.com/in/tej-chaute)
- All reports are attached as PDFs inside each case folder.
