# Case 03 — Seized

**Platform:** CyberDefenders  
**Difficulty:** Medium  
**Evidence:** Linux memory dump (dump.mem — 0.99 GB)  
**Case Number:** 2026-003  
**Date:** 26/03/2026  

---

## What was the case about?

A Linux CentOS 7 system was suspected to be compromised. No disk image — just a RAM dump. The entire investigation had to be done from memory using Volatility 2.

This was my first pure memory forensics case and the one I learned the most from.

---

## What I found

The attacker had done everything right from a persistence standpoint — which is exactly what made it interesting to investigate.

Initial access was through a reverse shell using `ncat` (PID 2854), connecting back to `192.168.49.1` on port `12345`. I found this through `linux_netstat` — the connection was still live in memory at the time of the dump. From there the attacker upgraded to a fully interactive shell using a Python one-liner (`python -c 'import pty; pty.spawn("/bin/bash")'`) — standard technique to make a reverse shell usable for real work.

The bash history recovered via `linux_bash` on PID 2887 showed the attacker pulling a base64-encoded payload from an external Pastebin URL via git clone. Decoded it and confirmed it was a message left by the attacker.

The persistence mechanisms were the most technically interesting part. The attacker added an RSA public key directly to `/home/k3vin/.ssh/authorized_keys` — which means even if someone noticed and changed k3vin's password, the attacker could still SSH straight back in. Password resets do not touch authorized_keys files. A lot of incident responders get caught off guard by this.

On top of that there was a kernel-level rootkit — `sysemptyrect` — detected through `linux_check_syscall`. It had hooked the syscall table to hide itself from the OS. The encryption key it used (`1337tibbartibbar`) came out of `linux_lsmod -P`. A rootkit hiding at the kernel level combined with SSH key persistence means the attacker was planning to be in this system for a very long time.

---

## Artifacts that mattered most

| Artifact | Volatility Plugin | What it showed |
|----------|------------------|----------------|
| Running processes | `linux_pstree`, `linux_pslist` | ncat running — reverse shell process identified (PID 2854) |
| Network connections | `linux_netstat` | Live C2 connection to 192.168.49.1:12345 |
| Shell activity | `linux_psaux` | Python TTY upgrade command in process args |
| Bash history | `linux_bash` | Encoded payload, git clone, attacker commands (PID 2887) |
| Memory extraction | `linux_dump_map` | SSH public key recovered from process memory |
| Rootkit detection | `linux_check_syscall` | sysemptyrect hooking the syscall table |
| Kernel modules | `linux_lsmod -P` | Rootkit encryption key identified |

---

## Volatility commands I ran

```bash
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_pstree
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_psaux
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_netstat
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_bash
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_dump_map --pid=2887 -D ./output/
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_check_syscall
vol.py -f dump.mem --profile=LinuxCentos7_3_10_0-514x64 linux_lsmod -P
strings dump.mem | grep -i 'centos' | head -20
```

---

## Tools used

| Tool | Version |
|------|---------|
| Volatility 2 | 2.6.1 |
| strings | 2.44 |
| grep | 3.11 |
| base64 | 9.5 |

---

## What I took away from this

The thing that stuck with me: changing a password does absolutely nothing if the attacker already has their SSH key in `authorized_keys`. I knew this conceptually before but seeing it in an actual investigation made it real in a way reading about it never did.

Some evidence only exists in RAM. The live network connection, the running ncat process, the bash history in memory — none of that would be on disk. If someone had rebooted this machine before taking the dump, that evidence would be gone forever. Memory forensics and disk forensics are not interchangeable — they show you completely different things, and you need both.

The rootkit was the hardest part to interpret. Knowing that `linux_check_syscall` flagged something was easy. Understanding what `sysemptyrect` was actually doing and why the encryption key mattered took more digging outside the tool output — which is where the real learning happened.

---

[View Full Report](./Seized-Report.pdf)
