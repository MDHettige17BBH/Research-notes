# Nmap Fundamentals

Cross-cutting technique note — not tied to one vulnerability class or one
machine. This is the base workflow every enumeration step starts from.

## What it is

Nmap is a network scanner. It tells you what's alive on a network and
what doors (ports/services) are open on it. Before you can attack, break,
or even understand a target, you need this picture first.

## Core scan types, in order of what you'd actually run

### 1. Basic scan — what's open?

```bash
nmap <TARGET_IP>
```

Checks the 1000 most common TCP ports. Tells you *which* ports respond —
nothing about what's actually running on them yet.

### 2. Version scan — what's running?

```bash
nmap -sV <TARGET_IP>
```

Same port check, but now it fingerprints the actual software and version
behind each open port. This matters because security depends on the
*exact* version — a port being open tells you almost nothing on its own;
knowing it's "Apache 2.4.49" tells you whether a known CVE applies.

### 3. Aggressive scan — the full picture

```bash
nmap -A <TARGET_IP>
```

Bundles OS detection, version detection, script scanning, and traceroute
into one scan. Slower, louder, but gives the fullest picture in one shot.
Good for a first pass on a box you're allowed to hit hard; too noisy for
anything where stealth matters.

### 4. Full port sweep — don't trust the default 1000

```bash
nmap -p- <TARGET_IP>
```

Scans all 65,535 TCP ports instead of just the common 1000. Slow (can
take minutes), but the default scan misses anything running on a
non-standard port — and deliberately hiding a service on a weird port is
a real thing people do.

## Reading the output

- **"Host is up"** — the target responded, Nmap reached it. Doesn't mean
  anything is open yet, just that something answered.
- **"996 filtered/closed"** — ports Nmap checked that didn't respond or
  actively refused. Normal, not an error.
- **A `?` next to a service name** (e.g. `21/tcp open ftp?`) — Nmap is
  guessing based on the port number convention, not confirmed by actual
  banner/response data. Don't trust it as fact — verify manually.

## Worked example — scanning my own PC (2026-07-19)

Confirming the tool works first:

```text
C:\Users\malik>nmap --version
Nmap version 7.98 ( https://nmap.org )
Platform: i686-pc-windows-windows
Compiled with: nmap-liblua-5.4.8 openssl-3.0.17 nmap-libssh2-1.11.1 nmap-libz-1.3.1 nmap-libpcre2-10.45 Npcap-1.83 nmap-libdnet-1.18.0 ipv6
Compiled without:
Available nsock engines: iocp poll select
```

Finding my own local IP before scanning anything:

```text
C:\Users\malik>ipconfig

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.43.4
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.43.1
```

> If connected via cable instead of Wi-Fi, look for "Ethernet adapter
> Ethernet" instead of "Wireless LAN adapter Wi-Fi" — different section,
> same idea.

Basic scan:

```text
$ nmap 192.168.1.25

Not shown: 996 filtered tcp ports (no-response)
PORT     STATE  SERVICE
21/tcp   open   ftp
554/tcp  open   rtsp
1723/tcp open   pptp
5060/tcp open   sip

Final times for host: srtt: 35176 rttvar: 9020  to: 100000
```

Scanned 2026-07-19 18:39:31 SLST, took 76s.

Version scan on the same target:

```text
$ nmap -sV 192.168.1.25

Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-19 18:42 +0530
Nmap scan report for 192.168.1.25
Host is up (0.045s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE  SERVICE  VERSION
21/tcp   open   ftp?
554/tcp  open   rtsp?
1723/tcp open   pptp?
5060/tcp open   sip?
```

Port breakdown from this scan:
- **554 (RTSP)** — Real Time Streaming Protocol
- **1723 (PPTP)** — Point-to-Point Tunneling Protocol, used for VPN connections
- **5060 (SIP)** — Session Initiation Protocol, used for VoIP/communication systems
- The `?` on every service here means Nmap is guessing from port number
  convention alone, not a confirmed banner response — matches the
  "don't trust it as fact" point above, in practice.

## Common ports worth having memorized

| Port | Service | Notes |
|---|---|---|
| 21 | FTP | File transfer, often unencrypted |
| 22 | SSH | Encrypted remote login — the secure replacement for Telnet |
| 23 | Telnet | Remote login, **plaintext**, effectively obsolete/insecure |
| 25 | SMTP | Sending email |
| 53 | DNS | Domain name lookups |
| 80 | HTTP | Web, unencrypted |
| 443 | HTTPS | Web, encrypted |
| 445 | SMB | Windows file sharing |
| 3306 | MySQL | Database access |
| 3389 | RDP | Windows remote desktop |

Know these cold — they're the first read on any scan result, before you
dig into anything version-specific.

## Mistake worth remembering

Before scanning anything, confirm the target IP is actually the intended
target — not your own gateway, not a neighboring device on the same
subnet. Ran into this directly: scanned `192.168.43.1` (the router)
thinking it was my own machine, because I hadn't checked `ipconfig`
output carefully enough first. The fix is procedural, not technical:

```text
1. Find your target IP
2. Confirm it belongs to the intended device
3. Scan
```

Skipping step 2 wastes time and, on someone else's network, could cross
into scanning something you were never authorized to touch.

## Related

- Every Tier-0 HTB writeup so far starts with one of these scans —
  see `Bug-bounty-writeups/hack-the-box-academy/Tier-0/`
- `general-techniques/burp-workflow.md` — the equivalent base workflow
  once you're past network recon and into an actual web application

## Other scan types worth knowing

Beyond the four core scans above, these come up often enough to know
by name — not daily-driver commands, but recognize them when you see
them in someone else's writeup or a program's testing notes:

```bash
nmap -sX <TARGET_IP>      # XMAS scan — sets FIN, PSH, URG flags
nmap -sF <TARGET_IP>      # FIN scan — sends only the FIN flag
nmap -sN <TARGET_IP>      # NULL scan — no flags set at all
nmap -sA <TARGET_IP>      # ACK scan — used to map firewall rules, not find open ports
nmap -iL targets.txt      # scan a list of targets from a file
nmap --script <script>    # run a specific NSE (Nmap Scripting Engine) script
nmap --top-ports 100      # scan just the N most common ports — faster, less thorough
```

**Why XMAS/FIN/NULL exist:** they're designed to slip past older or
poorly configured firewalls that only filter based on the SYN flag,
since these scans never send one. Against a modern stateful firewall
they're mostly ineffective and often just get logged as noise — but
worth recognizing since they show up in older writeups and some CTF
contexts.

**ACK scans don't tell you if a port is open** — they tell you whether
a firewall is filtering it at all. Different question entirely from the
scans above; used for firewall rule mapping, not service discovery.
