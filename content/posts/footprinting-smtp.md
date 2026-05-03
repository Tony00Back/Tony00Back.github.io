---
title: "Footprinting — SMTP"
date: 2026-05-02
tags: ["SMTP", "Footprinting", "CPTS", "HTB", "Email"]
categories: ["HTB Academy"]
series: ["Footprinting"]
description: "SMTP enumeration — user enumeration via VRFY, open relay detection, and what email servers leak."
showToc: true
---

## Overview

SMTP was designed in the 1980s for a small, trusted internet. Authentication was optional, relay was open by default. Three decades later, those design decisions are still causing security issues.

## The Email Flow

```
[You - MUA]  →  MSA (port 587)  →  MTA (port 25)  →  MTA (port 25)  →  MDA  →  Mailbox
               (auth required)    (your server)      (dest server)
```

| Component | Name | Role |
|-----------|------|------|
| MUA | Mail User Agent | Your email client |
| MSA | Mail Submission Agent | Validates and accepts from client |
| MTA | Mail Transfer Agent | Server-to-server delivery |
| MDA | Mail Delivery Agent | Drops into recipient mailbox |

## Ports

| Port | Use | Encryption |
|------|-----|------------|
| 25 | MTA ↔ MTA | None by default |
| 587 | Client → Server | STARTTLS (upgrade) |
| 465 | Client → Server | SSL/TLS direct |

## Enumeration

### Banner Grabbing
```bash
telnet <IP> 25
```
The banner reveals: hostname, software (Postfix/Sendmail/Exim), version, OS.

### EHLO — List Capabilities
```bash
EHLO attacker.com
```
Look for: `VRFY` (user enumeration), `AUTH PLAIN` (auth method), `STARTTLS` (encryption support).

### User Enumeration via VRFY
```bash
VRFY root
VRFY admin
```
`250` = user exists. `550` = doesn't exist. `252` = server won't confirm (unreliable).

### Automated User Enumeration
```bash
smtp-user-enum -M VRFY -U /path/to/wordlist.txt -t <IP> -w 15
```
> **Key:** `-w 15` sets the timeout. If getting 0 results, the server is slow — increase wait time before switching methods.

### Open Relay Check
```bash
nmap -p25 --script smtp-open-relay -v <IP>
```
An open relay (`mynetworks = 0.0.0.0/0`) lets anyone send email as anyone. Classic phishing vector.

## Key Misconfigurations

**`mynetworks = 0.0.0.0/0`** — Open relay. Send email from any IP as any sender.

**`VRFY` enabled** — Enumerate valid system users.

**Verbose banner** — Software version → CVE research.

## What SMTP Exposes

- Internal hostnames from banner
- Valid usernames (for password attacks)
- Email infrastructure layout
- Potential phishing vector via open relay
