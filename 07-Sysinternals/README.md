# Case 07 — Sysinternals

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics  
**Difficulty:** Medium (Retired)  
**Evidence:** Windows disk image — `SysInternals.E01` (2.98 GB)  
**Case Number:** 2026-007  

---

## Scenario

A user thought they were downloading the Sysinternals tool suite. The tools never launched, and the system started slowing down afterward. A disk image was taken and handed over for analysis — figure out what actually got downloaded, whether it ran, and what it did.

---

## What I Found

The interesting part of this case wasn't what was there — it was what was missing. There were no Prefetch files for `Sysinternals.exe`. If I'd stopped there I would've written "no evidence of execution" and closed it. But that would've been wrong.

ShimCache had an entry for the file. AmCache had an entry with an actual execution timestamp and a SHA1 hash. Two independent sources saying the file ran, and neither of them is Prefetch. That's the core lesson of this whole investigation — if one artifact is absent, you keep looking.

The file itself was gone by the time I was looking at the image. Deleted. But the SHA1 hash was preserved in AmCache, which meant I could pivot to VirusTotal and get a clean identification: **Rozena malware family**, classified as a downloader.

PowerShell history filled in the rest. Before the malware even executed, someone ran commands to modify the hosts file — mapping `www.malware430.com` and `www.sysinternals.com` to an internal IP (`192.168.15.10`). So any traffic to those domains would get silently redirected. The fact that this happened two minutes *before* the malware ran raises a question about sequencing that I documented in the notes.

VirusTotal's behavioural analysis showed the rest: the initial payload spawned `cmd.exe`, which dropped a second executable called `vmtoolsIO.exe`. That second stage installed a Windows service named `VMwareIOHelperService` — configured to autostart, named to look like a legitimate VMware component. The kind of name an analyst might glance at and move on from.

---

## Artifacts Documented

| # | Artifact | Key Finding |
|---|----------|-------------|
| 1 | Executable file | `Sysinternals.exe` downloaded to `C:\Users\Public\Downloads\` — later deleted |
| 2 | ShimCache | Entry confirms file was registered by OS compatibility layer; timestamp = last modified time of file (21:18:51), not execution time |
| 3 | AmCache | Execution confirmed at 21:18:00; SHA1 hash preserved post-deletion — authoritative identifier |
| 4 | VirusTotal | SHA1 hash matched Rozena malware family — downloader classification |
| 5 | PowerShell history | Hosts file modified at 21:17:03 — two minutes before execution — redirecting malicious and spoofed domains to 192.168.15.10 |
| 6 | Hosts file | `C:\Windows\System32\drivers\etc\hosts` modified to map attacker domains to internal IP |
| 7 | VirusTotal process tree | Malware spawned `cmd.exe` → `vmtoolsIO.exe` — multi-stage dropper confirmed |
| 8 | VirusTotal behavioural | `VMwareIOHelperService` installed as autostart Windows service — persistence via masquerading |

---

## What I Learned

**Prefetch absence is not proof of non-execution.** This sounds obvious in hindsight but it's easy to treat a missing artifact as evidence of nothing happening. The right move is to treat it as a gap and keep checking. ShimCache and AmCache both exist precisely because Prefetch isn't always available — Windows 10 Enterprise disables Prefetch by default on SSDs.

**ShimCache timestamps are not execution timestamps.** The entry for `Sysinternals.exe` showed `21:18:51` — but that's the file's last modified time, not when it ran. Execution time came from AmCache: `21:18:00`, nearly a minute earlier. Getting these confused would've produced an incorrect timeline.

**A deleted file can still be fully identified.** The executable was gone, but AmCache had the SHA1 hash. That hash went straight into VirusTotal and came back with a clean family identification. No file needed. This is why AmCache matters in malware investigations — it outlives the sample.

**Service names are not trustworthy.** `VMwareIOHelperService` looks reasonable if you're scanning a service list quickly. The name was chosen specifically to blend in. Verifying service legitimacy — checking the binary path, the publisher, whether it's expected on that machine — needs to be a habit rather than an afterthought.

---

[View Full Report](./Sysinternals-Report.pdf)
