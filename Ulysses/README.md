# Case 04 — Ulysses

**Platform:** CyberDefenders
**Difficulty:** Medium
**Evidence:** Linux disk image (victoria-v8.sda1.img — 909 MB) + memory dump (victoria-v8.memdump.img — 255 MB)
**Case Number:** 2026-004
**Date:** 01/04/2026

---

## What was the case about?

A research server was flagged after multiple failed authentication attempts and suspicious outbound connections. I was given both a disk image and a memory dump from the same machine and asked to reconstruct how the attacker got in, what they did, and how they maintained access.

This was my first case with both evidence types together — and the main challenge was knowing which source to use for which question.

---

## What I found

The attack happened in two stages, and if I had only looked at the first stage I would have missed how the system was actually compromised.

Stage one was an SSH brute force attack against a user account called `ulysses`, coming from `192.168.56.1`. I found this in `auth.log` on the disk image — hundreds of failed login attempts between 15:16 and 15:17 UTC. The brute force completely failed. No successful authentication at all.

Stage two is where it gets interesting. Two minutes after the brute force stopped, a different IP address (`192.168.56.101`) initiated an SMTP session with the Exim4 mail service. The attacker used a known Exim4 buffer overflow vulnerability to inject a `wget` command directly through SMTP, which downloaded a malicious archive called `rk.tar` to `/tmp/` on the victim machine. This is all sitting in `/var/log/exim4/mainlog` on disk — the command injection is visible in plain text in the log.

The memory dump is what confirmed what happened after that. A `netcat` process (PID 2169) was still running at the time the dump was taken, with an established TCP connection back to the attacker on port 8888. That connection — confirmed live in RAM — was the exfiltration channel. It would have been completely invisible from disk alone.

The extracted `rk.tar` archive contained an `install.sh` script that deployed a rootkit and added an iptables rule blocking outbound traffic on port 45295 — likely to prevent other attackers from using the same machine or to evade specific detection tools.

---

## Why both sources were necessary

This is the thing Ulysses really drives home. Some questions could only be answered from disk, and some only from memory:

| Question | Source needed |
|----------|--------------|
| What account was brute-forced? | Disk — auth.log |
| How did the attacker actually get in? | Disk — exim4 mainlog |
| What was downloaded and where? | Disk — /tmp/ filesystem |
| Was there an active backdoor? | Memory — linux_pslist |
| Was data being exfiltrated? | Memory — linux_netstat |
| What port was the C2 channel? | Memory — linux_netstat |

Neither source alone gives the full picture. That's the point.

---

## Artifacts that mattered most

| Artifact | Location / Source | What it showed |
|----------|------------------|----------------|
| SSH brute force attempts | `/var/log/auth.log` (disk) | Failed logins for user 'ulysses' from 192.168.56.1 |
| DHCP lease | `/var/lib/dhcp3/dhclient.eth0.leases` (disk) | Victim IP confirmed as 192.168.56.102 |
| Exim4 SMTP interaction | `/var/log/exim4/mainlog` (disk) | Attacker session from 192.168.56.101 at 15:19 |
| Command injection | `/var/log/exim4/mainlog` (disk) | wget command embedded in SMTP — remote code execution confirmed |
| Payload archive | `/tmp/rk.tar` (disk) | Rootkit installer downloaded to system |
| Netcat process | `linux_pslist` — PID 2169 (memory) | Active backdoor process running |
| C2 connection | `linux_netstat` (memory) | Live connection to 192.168.56.1:8888 confirmed |
| Rootkit installer | `rk.tar` → `install.sh` (disk) | Persistence mechanisms and iptables modification |

---

## Volatility commands I ran

```bash
vol.py --profile=LinuxDebian5_26x86 -f victoria-v8.memdump.img linux_banner
vol.py --profile=LinuxDebian5_26x86 -f victoria-v8.memdump.img linux_pslist
vol.py --profile=LinuxDebian5_26x86 -f victoria-v8.memdump.img linux_psaux
vol.py --profile=LinuxDebian5_26x86 -f victoria-v8.memdump.img linux_netstat
vol.py --profile=LinuxDebian5_26x86 -f victoria-v8.memdump.img linux_bash

```

The custom profile `Debian5_26.zip` had to be installed to `volatility/plugins/overlays/linux/` before any of these would work.

---

## Tools used

| Tool | Version |
|------|---------|
| Autopsy | 4.22.1 |
| Volatility 2 | 2.6.1 |
| strings | 2.46 |
| grep | 3.12 |

---

## What I took away from this

The SSH brute force was loud and obvious — hundreds of log entries, easy to find. The actual compromise was quiet — a few lines in an obscure mail server log that look almost normal until you read them carefully. That mismatch is intentional on the attacker's side. Generate noise in one place while doing the real work somewhere else.

The other thing: I initially flagged a timestamp discrepancy between `rk.tar`'s creation time and the exim4 log as potential evidence tampering. It turned out to be a UTC offset. That was a useful reminder — always look for the mundane explanation before the malicious one. If the numbers align after a timezone adjustment, it is almost always just a timezone issue.

Memory-only evidence is the most important takeaway from this case. The live netcat connection to port 8888 would have been gone the moment someone rebooted the machine. The IR team acquired the dump while the system was still running — which is exactly the right call.

---

[View Full Report](./Ulysses-Report.pdf)
