# Case 05 — AfricanFalls

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics  
**Difficulty:** Medium (Retired)  
**Evidence:** Windows disk image — `DiskDrigger.ad1` (686 MB)  
**Case Number:** 2026-005  

---

## Scenario

John Doe was accused of conducting illegal activities. A disk image of his laptop was taken. The objective was to analyse the image as a SOC analyst and reconstruct what happened — covering browser activity, credential access, network behaviour, external device usage, and deleted files.

---

## What I Found

The investigation uncovered a clear sequence of suspicious behaviour:

- **Credential acquisition** — The user searched for password-cracking resources via Chrome, then deleted the downloaded list within five minutes — a deliberate anti-forensics action.
- **Tor Browser** — The installer was executed but prefetch evidence confirmed the browser itself was never launched. An important distinction.
- **FTP activity** — FileZilla configuration revealed a connection to an internal FTP server at `192.168.1.20`.
- **Network reconnaissance** — PowerShell history showed a port scan executed against the external domain `dfir.science`.
- **External device** — USBSTOR registry keys identified a connected LGE USB drive. EXIF metadata from an image on the device placed it in **Zambia**, corroborating external activity.
- **Credential recovery** — NTLM hashes extracted from the SAM hive via Mimikatz were cracked with Hashcat, recovering both a secondary password (`AFR1CA!`) and the Windows login password (`ctf202`).

---

## Artifacts Documented

| # | Artifact | Key Finding |
|---|----------|-------------|
| 1 | Chrome Browser History | Searched for "password cracking lists" |
| 2 | FileZilla Config | FTP connection to 192.168.1.20 |
| 3 | Recycle Bin `$I` file | Password list deleted shortly after download |
| 4 | Prefetch — Tor Browser installer | Installer ran; browser never launched |
| 5 | PowerShell ConsoleHost_history.txt | Port scan against dfir.science |
| 6 | EXIF Metadata | Image GPS coordinates → Zambia |
| 7 | USBSTOR Registry Key | LGE USB Drive identified by FriendlyName |
| 8 | SAM Registry Hive | NTLM hashes extracted for all local accounts |
| 9 | Hash Analysis | `3DE1A36F...` → `AFR1CA!` |
| 10 | Hash Analysis | `ecf53750...` → `ctf202` (Windows login) |

## What I Learned

**Prefetch as execution proof** — Prefetch files don't just show that an executable ran; they can also prove something *didn't* run. The Tor Browser installer created a prefetch entry, but no corresponding entry existed for `firefox.exe` (the actual browser). That absence is evidence.

**Hash chain documentation** — For credential findings, documenting the full chain (hash value → cracking tool → plaintext) matters. It makes the finding reproducible and auditable, which is the standard in real investigations.

**USBSTOR without the device** — The FriendlyName value in `SYSTEM\CurrentControlSet\Enum\USBSTOR` identified the exact device model (LGE USB Drive) without the physical hardware being present. Combined with EXIF GPS data from a photo on the device, this built a surprisingly complete picture of external activity.

**Timestamp gaps are findings too** — Some artifacts (USB copy times, certain registry writes) had no reliable timestamps. Documenting what *couldn't* be determined — and why — is part of professional reporting.


[View Full Report] (./Africanfalls-Report)
