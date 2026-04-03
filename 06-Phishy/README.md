# Case 06 — Phishy

**Platform:** CyberDefenders  
**Category:** Endpoint Forensics    
**Difficulty:** Medium (Retired)    
**Evidence:** Windows disk image — `GiveAway.ad1` (892 MB)    
**Case Number:** 2026-006    

---

## Scenario

A company employee fell for a fake iPhone giveaway. A disk image of their system was taken and handed to me to figure out how they got compromised — how the attacker made contact, what was delivered, and whether anything was stolen.

---

## What I Found

This one had more moving parts than the previous labs. The attack came in through WhatsApp, which immediately made it feel more realistic — most phishing investigations I'd read about use email, so having to dig through a WhatsApp database was new territory.

The attacker sent the victim a link disguised as an Apple giveaway. The domain was `appIe.com` — that capital "I" instead of a lowercase "l" is easy to miss at a glance, and that's exactly the point. The victim clicked, downloaded a Word document, and that's where things got worse.

The document had embedded VBA macros that ran the moment it was opened — no clicking "enable macros", no prompts, just `Document_Open` firing automatically. The macro was obfuscated (Base64 + chr() concatenation), so I had to pull it through CyberChef to see what it was actually doing. Once decoded: it was calling PowerShell to pull down a second file, `IPhone.exe`, from the same attacker domain and drop it in `C:\Temp`.

VirusTotal confirmed the exe was a Metasploit Meterpreter payload. The attacker's C2 server was at `155.94.69.27`.

On top of all that, the victim also visited a fake Apple login page at `appIe.competitions.com` and handed over their credentials. Firefox had saved them, so PasswordFox recovered them directly from `logins.json`.

---

## Artifacts Documented

| # | Artifact | Key Finding |
|---|----------|-------------|
| 1 | Registry — ComputerName | Hostname: WIN-NF3JQEU4G0T |
| 2 | Downloads — WhatsApp.exe | WhatsApp confirmed as delivery platform |
| 3 | WhatsApp msgstore.db | Attacker sent phishing URL via WhatsApp chat |
| 4 | Malicious URL | `http://appIe.com/IPhone-Winners.doc` — typosquatted Apple domain |
| 5 | Downloaded document | `IPhone-Winners.doc` — first-stage malware delivery |
| 6 | oledump stream 9 & 10 | Embedded VBA macros confirmed in document |
| 7 | VBA `Document_Open` | Macro auto-executes on file open — no user action needed |
| 8 | Decoded PowerShell | Obfuscated macro decodes to PowerShell download command |
| 9 | Payload URL | `http://appIe.com/Iphone.exe` — second-stage download |
| 10 | `C:\Temp\IPhone.exe` | Payload written to disk — staging confirmed |
| 11 | VirusTotal hash match | Payload = Metasploit Meterpreter binary |
| 12 | C2 IP: 155.94.69.27 | Outbound C2 connection confirmed via VirusTotal / Hybrid Analysis |
| 13 | Firefox places.sqlite | Victim visited fake Apple login page |
| 14 | Phishing URL | `http://appIe.competitions.com/login.php` |
| 15 | Firefox logins.json | Credentials saved after submission to phishing page |

---

---

## What I Learned

**WhatsApp as an attack vector is genuinely tricky to investigate.** The msgstore.db format isn't something you can just open and read. WhatsApp Viewer had a version compatibility issue with the database, so I ended up querying it directly in DB Browser for SQLite. Good reminder that forensic tools don't always cooperate and you need to be comfortable going one layer deeper.

**VBA macro obfuscation isn't as scary as it sounds.** The obfuscation here was chr() concatenation feeding into a Base64-encoded string. Once you recognise the pattern it's fairly mechanical to decode. CyberChef makes it straightforward — paste the Base64, hit "decode", get your payload URL. What matters is recognising *that* it's obfuscated in the first place, which oledump and olevba flag clearly.

**The C2 IP wasn't on disk.** There were no network logs, no PCAP, nothing in the image that directly recorded the outbound connection. I had to pivot to VirusTotal using the malware hash to find the C2 infrastructure. That's a realistic constraint — disk forensics alone won't always give you the full picture, and knowing when to use external threat intel is part of the job.

**Timeline anomalies are findings, not mistakes.** The credential timestamp being a day earlier than the document download initially looked like an error in my analysis. But when I sat with it, it actually told me something: the victim probably visited the login page directly from the WhatsApp link first, then came back to the document later. A weird timestamp is a clue, not a problem to explain away.

---

[View Full Report](./Phishy-Report)
