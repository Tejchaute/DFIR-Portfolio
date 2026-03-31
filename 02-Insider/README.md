# Case 02 — Insider

**Platform:** CyberDefenders  
**Difficulty:** Easy  
**Evidence:** Linux disk image (FirstHack.ad1 — 83.6 MB)  
**Case Number:** 2026-002  
**Date:** 23/03/2026  

---

## What was the case about?

Suspected insider threat on a Linux system. I was given a disk image and told to go through the logs and bash history to figure out what the user actually did.

This one was more straightforward than Hunter — FTK Imager gave me direct access to everything without needing terminal. But what I found was more blatant than I expected.

---

## What I found

The most surprising artifact was not a tool or a log entry — it was a literal checklist file sitting on the Desktop at `/root/Desktop/Checklist`. The user had written down their attack objectives. One of them was "Gain Bob's Trust." The third was "Profit." That is about as explicit as insider threat evidence gets.

Everything else confirmed the same picture. Mimikatz (`mimikatz_trunk.zip`) was downloaded to `/root/Downloads/` — a credential dumping tool with essentially no legitimate use on a personal workstation. Bash history showed Metasploit (`msfconsole`) being run, and `binwalk` being used on a suspicious image file (`didyouthinkwedmakeiteasy.jpg`) to extract hidden data.

There was also a JPEG (`irZLAohL.jpeg`) that contained a screenshot of `hping3` running — hping3 is a network packet crafting tool used for traffic generation and DoS attacks. That screenshot was the evidence that an actual attack had been launched, not just prepared.

The privilege escalation piece was interesting too. The `postgres` service account had been used to run `su` commands repeatedly on 2019-03-20, two days before everything else. The user had set up elevated access in advance before executing the main attack.

---

## Artifacts that mattered most

| Artifact | Path | What it showed |
|----------|------|----------------|
| Checklist file | `/root/Desktop/Checklist` | Documented attack goals — social engineering and financial motive |
| Downloaded tool | `/root/Downloads/mimikatz_trunk.zip` | Credential dumping tool staged on system |
| Bash history | `/root/.bash_history` | Metasploit execution, binwalk usage, file creation activity |
| Image evidence | `/root/irZLAohL.jpeg` | Screenshot of hping3 confirming active network attack |
| Auth logs | `/var/log/auth.log` | postgres user escalating to root two days before main activity |
| Bash script | `/root/Documents/firstscript_fixed` | Taunting message targeting individual named "Young" |

---

## Tools used

| Tool | Version | Note |
|------|---------|------|
| FTK Imager | 4.7.3.81 | All artifacts accessed directly via FTK file browser — no terminal needed |

---

## What I took away from this

The checklist file was the kind of thing you almost do not believe you found. Written evidence of intent is rare — usually you are inferring motivation from behavior patterns. Having it spelled out made the conclusion easy to write but also made me think about how often people leave behind things like this without realizing it.

The two-day gap between the postgres escalation and the main attack activity stood out as a pattern I had not consciously thought about before — preparation phase followed by execution phase. That sequencing matters a lot when building a timeline and arguing premeditation.

---

[View Full Report](./Insider-Report.pdf)
