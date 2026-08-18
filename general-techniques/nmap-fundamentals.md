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
