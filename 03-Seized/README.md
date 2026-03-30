# Case 03 — Seized

**Platform:** CyberDefenders   
**Category:** Memory Forensics   
**Difficulty:** Medium  
**Evidence:** Linux memory dump (dump.mem — 0.99 GB)  
**Case Number:** 2026-003  
**Date:** 26/03/2026  

---

## Scenario

A memory forensics investigation was conducted on a compromised Linux CentOS 7 system after suspicious activity was detected. A full memory dump was acquired to analyze running processes, network connections, bash history, and potential rootkits.

The objective was to determine how the attacker gained access, what actions were performed, and how persistence was maintained.

---

## What I Found

The investigation revealed a **multi-stage system compromise** involving remote access, payload delivery, persistence mechanisms, and kernel-level stealth techniques.

Key findings include:

- **Initial access via reverse shell** — attacker used `ncat` to establish a connection to a remote system
- **Interactive shell upgrade** — Python `pty.spawn` used to convert shell into a fully interactive TTY
- **Active C2 communication** — live TCP connection identified (192.168.49.1:12345)
- **Encoded command execution** — base64 payload found in bash history
- **Payload delivery** — attacker cloned a repository and fetched payload from Pastebin
- **Persistence via SSH key injection** — RSA key added to `/home/k3vin/.ssh/authorized_keys`
- **Kernel-level rootkit detected** — `sysemptyrect` modified syscall table
- **Advanced obfuscation** — encryption key `1337tibbartibbar`

These findings confirm a **fully compromised system with long-term persistence and stealth capabilities**.

---

## Artifacts Investigated

| Artifact Type        | Location / Source          | Key Finding                                      |
|----------------------|----------------------------|--------------------------------------------------|
| OS Information       | strings + grep             | CentOS Linux 7.7.1908 identified                 |
| Running processes    | linux_pslist, linux_pstree | Suspicious `ncat` (PID 2854)                     |
| Bash history         | linux_bash                 | Encoded payload + attacker commands              |
| Network connections  | linux_netstat              | Reverse shell connection                         |
| Shell activity       | linux_psaux                | Python TTY upgrade                               |
| Memory extraction    | linux_dump_map             | SSH key injection                                |
| Rootkit detection    | linux_check_syscall        | Syscall table hooked                             |
| Kernel modules       | linux_lsmod                | Rootkit + encryption key                         |

---

## Tools Used

| Tool         | Version | Purpose                                      |
|--------------|--------|----------------------------------------------|
| Volatility 2 | 2.6.1  | Memory analysis                              |
| strings      | 2.44   | Extract raw strings                          |
| grep         | 3.11   | Filter artifacts                             |
| base64       | 9.5    | Decode payload                               |

---

## Event Timeline

| Seq | Phase                | Event Description |
|-----|---------------------|------------------|
| 1   | Initial Access      | Reverse shell via `ncat` |
| 2   | Execution           | Shell upgraded using Python TTY |
| 3   | Discovery           | Base64 command executed |
| 4   | Payload Delivery    | Git clone + Pastebin payload |
| 5   | Persistence (SSH)   | RSA key injected |
| 6   | Persistence (Rootkit)| Kernel rootkit installed |
| 7   | Post-Compromise     | Full system compromise |

---

## Key Takeaway

This case shows how attackers combine:

- Remote access (reverse shell)
- Payload execution
- Persistence (SSH keys)
- Advanced stealth (kernel rootkits)

**Key lesson:** Memory forensics is critical — many artifacts (like rootkits and live connections) are not visible on disk.

The combination of **SSH persistence + kernel rootkit** indicates a **highly skilled attacker with long-term access goals**.

---

## Full Report

[View Full Report](./Seized-Report.pdf)
