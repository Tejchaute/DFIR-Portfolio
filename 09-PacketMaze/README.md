# Case 09 — PacketMaze

**Platform:** CyberDefenders  
**Category:** Network Forensics  
**Difficulty:** Medium (Retired)  
**Evidence:** Network capture — `UNODC-GPC-001-003-JohnDoe-NetworkCapture-2021-04-29.pcap` (36.7 MB)  
**Case Number:** 2026-009  

---

## Scenario

A network capture was taken of suspicious internal user activity. The objective was to go through the traffic and determine what the user was doing — what services they accessed, what they transferred, and whether any of it was worth escalating.

---

## What I Found

This one was different from WebStrike. There was no obvious attacker, no exploitation, no web shell. Just a user doing things over a network and the question of whether those things were suspicious. That ambiguity made it harder in some ways — you're building a case for scrutiny rather than confirming a compromise.

The user on `192.168.1.26` authenticated to an internal FTP server using cleartext credentials — username `kali`, password `AfricaCTF2021` — both sitting in plain text in the packet stream. Anyone on the network could have read those. Once logged in, they browsed `/home/kali`, uploaded a JPG file, and then retrieved a ZIP called `accountNum.zip`. That filename is what pushed this from "routine FTP session" to "worth investigating further." You can't confirm what's in a file just from the name, but in a DFIR context, a file named `accountNum.zip` being pulled over cleartext FTP is the kind of thing you escalate rather than ignore.

The EXIF metadata on the uploaded JPG identified the originating device as an **LG LM-Q725K** smartphone. So whatever was in that image came from a mobile device — useful for attribution if this were a real investigation.

After the FTP session there was a DNS query for `www.7-zip.org` — consistent with someone researching or downloading archiving software. Then a TLS handshake with `protonmail.com`. ProtonMail isn't inherently suspicious, but combined with everything else — cleartext FTP, account-named ZIP file, archiving tool lookup — it fits a pattern of someone who is privacy-conscious about their outbound communications while being careless about internal ones.

I also extracted the **TLS client random** from the Client Hello packet. It won't help now, but if the server's private key were ever obtained, that value combined with the server random would let you decrypt the full session in Wireshark. Worth documenting.

The FTP server's MAC address resolved to **PCS Systemtechnik GmbH** — the OUI associated with VirtualBox virtual machines. So the "FTP server" was a VM, which is useful context for characterising the internal environment.

---

## Artifacts Documented

| # | Artifact | Key Finding |
|---|----------|-------------|
| 1 | FTP Authentication | USER kali / PASS AfricaCTF2021 — cleartext credentials exposed |
| 2 | FTP Directory Listing | `/home/kali` enumerated — interactive session confirmed |
| 3 | FTP File Upload | `20210429_152157.jpg` uploaded to FTP server |
| 4 | EXIF Metadata | Originating device identified as LG LM-Q725K smartphone |
| 5 | DNS Query | `www.7-zip.org` — archiving tool lookup |
| 6 | TLS Handshake | SNI: `protonmail.com` — encrypted session initiated |
| 7 | TLS Client Random | Extracted from Client Hello — session identifiable for future decryption |
| 8 | HTTP Request | GET to `dfir.science` — external web access confirmed |
| 9 | HTTP Redirect | Server redirected to HTTPS |
| 10 | MAC Address | FTP server MAC `08:00:27:a6:1f:86` → PCS Systemtechnik GmbH (VirtualBox) |
| 11 | FTP File Retrieval | `accountNum.zip` retrieved — filename suggests sensitive financial content |
| 12 | FTP Directory | Folder named `ftp` present since 2021-04-20 — ten days pre-investigation |

---

## Network Context

| Field | Value |
|-------|-------|
| Internal Host | 192.168.1.26 |
| FTP Server | 192.168.1.20 |
| FTP Mode | Passive (PASV) |
| FTP Credentials | USER kali / PASS AfricaCTF2021 (cleartext) |
| MAC Vendor | PCS Systemtechnik GmbH — VirtualBox VM |
| External Service | dfir.science |
| Encrypted Service | protonmail.com |

---

## What I Learned

**Ambiguity is harder to work with than a clear attack.** WebStrike was obvious — attacker, exploit, payload, done. This case required holding a more uncertain conclusion. The activity *could* be innocent. It could also not be. Documenting that uncertainty honestly, rather than forcing a definitive verdict, is what a real analyst would do.

**Passive FTP requires following two streams.** The control channel on port 21 handles commands. The actual file transfer happens on a separate negotiated port. If you only filter `tcp.port == 21` in Wireshark, you'll see the commands but miss the file contents entirely. I had to follow both streams to reconstruct the full session.

**TLS doesn't make destinations invisible.** The SNI field in the Client Hello leaks the hostname in plaintext, even over an encrypted connection. `protonmail.com` was visible despite TLS. And the client random is worth capturing even when you can't decrypt the session — if a private key becomes available later, that value is what makes decryption possible.

**The TLS client random value uniquely identifies this specific TLS session. If the server's private key were obtained, this value — combined with the server random — would allow the full session to be decrypted using tools such as Wireshark's TLS decryption feature. Documents the session for potential future decryption if keys become available.** Documenting the client random is a habit worth building.

**MAC address vendor lookup is a fast way to characterise infrastructure.** Finding out the FTP server was a VirtualBox VM took about ten seconds. In a real investigation that context — "this is a VM, not a physical server" — affects how you think about what else might be running on the same host.


---

[View Full Report](./PacketMaze-Report.pdf)
