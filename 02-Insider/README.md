# Case 02 — Insider

**Platform:** CyberDefenders
**Category:** Endpoint Forensics
**Difficulty:** Easy
**Evidence:** Linux disk image (FirstHack.ad1 — 83.6 MB)
**Case Number:** 2026-002
**Date:** 23/03/2026

---

## Scenario

An investigation was conducted on a Linux-based system following suspicion of insider misuse. A forensic disk image was acquired to analyze system artifacts — including logs and Bash history — to reconstruct user activity and determine whether malicious actions were performed.

---

## What I Found

The investigation confirmed **premeditated insider threat activity** involving privilege escalation, offensive tooling, and documented malicious intent.

Key findings include:

- **Privilege escalation** — user `postgres` repeatedly executed `su` commands to gain root access (early-stage preparation)
- **Credential theft preparation** — `mimikatz_trunk.zip` downloaded, indicating intent to extract credentials
- **Offensive tooling usage** — Metasploit (`msfconsole`) executed; binwalk used for hidden data extraction
- **Active attack execution** — hping3 used to generate crafted network traffic (confirmed via screenshot artifact)
- **File staging activity** — creation of `SuperSecretFile.txt` on Desktop via bash history
- **Explicit malicious intent** — a Checklist file documented goals such as social engineering (“Gain Bob’s Trust”) and financial motive (“Profit”)
- **Targeted behavior** — bash script contained a taunting message directed at a specific individual (“Young”)

The presence of a written checklist combined with tool usage and privilege escalation confirms this was not accidental activity but a **deliberate and structured insider attack**.

---

## Artifacts Investigated

| Artifact Type    | Location / Path                      | Key Finding                                  |
| ---------------- | ------------------------------------ | -------------------------------------------- |
| Boot files       | `/boot`                              | Confirmed Kali Linux environment             |
| Downloaded tool  | `/root/Downloads/mimikatz_trunk.zip` | Credential dumping tool staged               |
| Bash history     | `/root/.bash_history`                | Command execution, file creation, tool usage |
| Checklist file   | `/root/Desktop/Checklist`            | Explicit malicious objectives documented     |
| Image evidence   | `/root/irZLAohL.jpeg`                | Screenshot of hping3 attack activity         |
| Bash script      | `/root/Documents/firstscript_fixed`  | Targeted message to individual “Young”       |
| Auth logs        | `/var/log/auth.log`                  | Privilege escalation via postgres user       |
| Suspicious image | `/root/didyouthinkwedmakeiteasy.jpg` | Used with binwalk for hidden data extraction |

---

## Tools Used

| Tool       | Version  | Purpose                                    |
| ---------- | -------- | ------------------------------------------ |
| FTK Imager | 4.7.3.81 | AD1 image mounting and artifact extraction |

---

## Key Takeaway

This case highlights how insider threats often involve **early-stage preparation followed by execution**.

Key lesson: **correlating multiple artifact sources is critical** — Bash history, logs, and filesystem artifacts together revealed the full attack chain.

The most significant finding was the **Checklist file**, which explicitly documented malicious intent. Combined with privilege escalation and offensive tool usage, this confirms a **premeditated insider attack with clear objectives and technical capability**.

---

## Full Report

[View Full Report](./Insider-Report.pdf)
