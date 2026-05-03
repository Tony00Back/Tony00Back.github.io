---
title: "Footprinting — DNS"
date: 2026-05-01
tags: ["DNS", "Footprinting", "CPTS", "HTB"]
categories: ["HTB Academy"]
series: ["Footprinting"]
description: "DNS enumeration techniques — zone transfers, subdomain brute forcing, and what misconfigured DNS servers expose."
showToc: true
---

## Overview

DNS is one of the most valuable services for a pentester during the reconnaissance phase. A misconfigured DNS server can expose the entire internal network map — hostnames, IPs, domain controllers, mail servers, and VPNs — without triggering a single port scan.

## Key Concepts

### DNS Server Types

| Type | Role |
|------|------|
| Root Server | 13 globally, last resort for TLD resolution |
| Authoritative NS | Holds the zone, answers with binding data |
| Non-authoritative NS | Collects info via recursive/iterative queries |
| Caching Server | Stores results per TTL |
| Forwarding Server | Passes queries to another server |
| Resolver | Local resolution (your OS, your router) |

### Important DNS Records

| Record | Purpose |
|--------|---------|
| A | IPv4 address |
| AAAA | IPv6 address |
| MX | Mail server |
| NS | Name servers for the domain |
| TXT | SPF, DMARC, domain verification |
| CNAME | Alias for another domain |
| PTR | Reverse lookup (IP → hostname) |
| SOA | Zone authority and admin contact |

## Methodology

### 1. NS Query — Find the name servers
```bash
dig ns <domain> @<IP>
```

### 2. ANY Query — Dump all records
```bash
dig any <domain> @<IP>
```

### 3. Zone Transfer — Get the full zone file
```bash
dig axfr <domain> @<IP>
```

If AXFR works, you get every host in that zone instantly. No brute force needed.

### 4. Enumerate subdomains as separate zones
Subdomains like `dev`, `internal`, `app` can be independent zones with their own records:
```bash
dig axfr internal.<domain> @<IP>
dig axfr dev.<domain> @<IP>
```

### 5. Subdomain brute force (when AXFR fails)
```bash
dnsenum --dnsserver <IP> --enum -p 0 -s 0 \
  -f /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt \
  <domain>
```

> **Tip:** Use `fierce-hostlist.txt` for internal/corporate hostnames. Standard subdomains wordlists miss legacy names like `win2k`, `fileserver`, `backup01`.

## Key Misconfigurations

**`allow-transfer { any; };`** — Anyone can request the full zone file. One `dig axfr` gives you the complete internal network map.

**Exposed internal zones** — `internal.company.com` configured as a separate zone but with the same misconfigured AXFR policy.

## What DNS Exposes

A successful zone transfer gives you:
- All internal hostnames and IPs
- Domain Controller names (`dc1`, `dc2`)
- Mail servers
- VPN endpoints
- Legacy/forgotten systems

This is passive reconnaissance — no exploit, no payload, just a protocol working as designed but misconfigured.
