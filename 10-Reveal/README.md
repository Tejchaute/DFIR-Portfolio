# Case 10 — Reveal

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics — Memory  
**Difficulty:** Easy (Retired)  
**Evidence:** Windows memory dump — `192-Reveal.dmp` (2.00 GB)  
**Case Number:** 2026-010  

---

## Scenario

A SIEM alert flagged suspicious activity on a Windows 10 machine. A full memory dump was taken and handed over. Figure out what happened.

---

## What I Found

This was my first Windows memory forensics case with Volatility 3, and the attack it contained was genuinely interesting — a fileless intrusion that barely touched disk at all.

The process list looked mostly normal at first. Then I spotted `powershell.exe` (PID 3692) running at an unusual time. I ran `psscan` to look at process relationships and found something that `pslist` had completely missed: both `powershell.exe` *and* `wordpad.exe` had been spawned by the same parent — PID 4120 — which was no longer present in memory. A process that launched two children and then vanished. That's a dropper.

The wordpad.exe was a decoy. Launch something recognisable alongside your malicious process and analysts scanning a process list might stop investigating that branch. I didn't.

`cmdline` on PID 3692 showed what PowerShell was actually doing: `-windowstyle hidden` to keep the window invisible, then `net use` to mount a remote WebDAV share at `\\45.9.74.32\davwwwroot`, then `rundll32.exe` to execute `3435.dll` directly from that share. No file written to disk. The payload came in over the network, ran in memory, and left almost nothing behind for a disk-based scanner to find.

`netscan` showed an active outbound TCP connection to `45.9.74.32:8888` under `net.exe` (PID 2416). `net.exe` is a Windows utility for SMB and share management — it has no legitimate reason to be making raw TCP connections to external IPs on non-standard ports. That anomaly is worth flagging regardless of whether you already know the IP is bad.

`malfind` on PID 3692 found memory regions with `PAGE_EXECUTE_READWRITE` permissions containing shellcode patterns. RWX regions in a process that has no reason to be self-modifying confirm reflective DLL loading — the DLL was executing entirely in memory.

`getsids` confirmed the process was running under user **Elon** with administrative privileges. Everything the attacker did, they did with full admin rights.

The TTP pattern — WebDAV delivery, hidden PowerShell, rundll32 LOLBin, reflective DLL loading — matches the **StrelaStealer** malware family.

---

## Artifacts Documented

| # | Artifact | Volatility Plugin | Key Finding |
|---|----------|-------------------|-------------|
| 1 | Malicious process | windows.pslist | `powershell.exe` PID 3692 running at time of compromise |
| 2 | Process relationship | windows.psscan | PID 3692 and wordpad.exe both spawned by transient PID 4120 — dropper behaviour |
| 3 | Command execution | windows.cmdline | PowerShell run with `-windowstyle hidden`, `net use`, `rundll32` |
| 4 | Remote share access | windows.cmdline + strings | WebDAV share `\\45.9.74.32\davwwwroot` mounted for payload delivery |
| 5 | Payload execution | windows.pstree | `3435.dll` executed via `rundll32.exe` — LOLBin abuse |
| 6 | Network connection | windows.netscan | `net.exe` (PID 2416) with outbound TCP to `45.9.74.32:8888` — anomalous C2 |
| 7 | User context | windows.getsids | Process running under user Elon with admin privileges |
| 8 | Memory injection | windows.malware.malfind | RWX memory regions with shellcode — reflective DLL loading confirmed |

---

## Volatility 3 Commands Used

```
python3 vol.py -f 192-Reveal.dmp windows.info
python3 vol.py -f 192-Reveal.dmp windows.pslist
python3 vol.py -f 192-Reveal.dmp windows.pstree
python3 vol.py -f 192-Reveal.dmp windows.psscan
python3 vol.py -f 192-Reveal.dmp windows.cmdline
python3 vol.py -f 192-Reveal.dmp windows.getsids --pid 3692
python3 vol.py -f 192-Reveal.dmp windows.malware.malfind --pid 3692
python3 vol.py -f 192-Reveal.dmp windows.netscan
strings 192-Reveal.dmp | grep 45.9.74.32
```

---

## What I Learned

**pslist and psscan answer different questions.** `pslist` reads the active process list — it only shows processes currently linked in the EPROCESS doubly-linked list. `psscan` scans raw memory for EPROCESS structures regardless of whether they're linked. A process that has exited or been deliberately unlinked will show up in `psscan` but not `pslist`. Running both isn't redundant — it's essential. PID 4120 would have been invisible if I'd only used `pslist`.

**The wordpad.exe decoy is a real technique.** Spawning a benign-looking process alongside a malicious one creates visual noise in a process list. Someone doing a quick check might see `wordpad.exe`, assume a user opened a document, and move on. The malicious sibling doesn't get looked at. Being deliberate about investigating *all* children of a suspicious parent is the counter.

**net.exe making a TCP connection to port 8888 on an external IP is wrong.** Full stop. `net.exe` manages SMB shares, user accounts, and local network resources. It doesn't initiate raw TCP sessions to external addresses. The process-to-connection pairing in `netscan` output was the tell — not just the destination IP, but the combination of which process owned that socket.

**No file on disk doesn't mean no evidence.** The entire payload lived in memory. malfind found the RWX regions, cmdline showed where the DLL came from, netscan showed where it was calling back to. Disk forensics alone would have come back clean. Memory is where fileless attacks live.


---

[View Full Report](./Reveal-Report.pdf)
