# 01 — Nmap Network Reconnaissance

## Objective
Perform comprehensive network reconnaissance on a home network using Nmap
to discover live hosts, enumerate open ports, fingerprint services, and
identify potential vulnerabilities.

## Environment
- **Attacker Machine:** Parrot OS Security Edition (bare metal) — `192.168.100.60`
- **Target Network:** `192.168.100.0/24` (own home network)
- **Primary Target:** Huawei Home Gateway — `192.168.100.1`
- **Tool:** Nmap 7.95

---

## Methodology

### Phase 1 — Host Discovery
```bash
sudo nmap -sn 192.168.100.0/24
```
**Result:** 2 live hosts found out of 256 possible addresses.

| Host | Status | MAC Address | Vendor |
|------|--------|-------------|--------|
| 192.168.100.1 | UP | 78:58:60:A4:40:30 | Huawei Technologies |
| 192.168.100.60 | UP | — | Local machine (attacker) |

---

### Phase 2 — Port Scanning
```bash
sudo nmap 192.168.100.1
```
**Result:** 5 ports of interest identified.

| Port | State | Service |
|------|-------|---------|
| 21/tcp | filtered | FTP |
| 22/tcp | filtered | SSH |
| 23/tcp | **open** | **Telnet** ⚠️ |
| 53/tcp | open | DNS |
| 80/tcp | open | HTTP |

---

### Phase 3 — Service Version Detection
```bash
sudo nmap -sV 192.168.100.1
```
**Result:** Services and versions identified.

| Port | Service | Version |
|------|---------|---------|
| 23/tcp | Telnet | Huawei Home Gateway telnetd |
| 53/tcp | DNS | — |
| 80/tcp | HTTPS | ssl/http (Huawei web interface) |

**Device Type:** Broadband router

---

### Phase 4 — Aggressive Scan (OS + Traceroute)
```bash
sudo nmap -A 192.168.100.1
```
**Result:**
- **OS Detected:** Linux 3.5 (embedded Linux on router)
- **SSL Certificate:** Issued to Huawei Technologies Co., Ltd
  - Valid from: 2014-12-05
  - **Expired: 2024-12-04** ⚠️ (over 5 months expired)
- **Network Distance:** 1 hop (direct connection)

---

### Phase 5 — Targeted Port Scan Across Full Network
```bash
sudo nmap -p 22,80,443,21,23,3306,8080 192.168.100.0/24
```
**Result:** 4 live hosts discovered.

| Host | Notable Open Ports | Notes |
|------|--------------------|-------|
| 192.168.100.1 | 23, 80 | Huawei router |
| 192.168.100.3 | None | Unknown device (MAC: 36:1F:7D:24:7C:63) |
| 192.168.100.60 | None | Attacker machine (this machine) |
| 192.168.100.63 | None | Unknown device |

---

### Phase 6 — Vulnerability Scan (NSE Scripts)
```bash
sudo nmap --script vuln 192.168.100.1
```

---

## Vulnerabilities Identified

### CVE-2014-3566 — SSL POODLE (CONFIRMED VULNERABLE)
- **Severity:** High
- **Port:** 80/tcp (ssl/http)
- **Description:** The SSL protocol 3.0 uses nondeterministic CBC padding, enabling
  man-in-the-middle attackers to obtain cleartext data via a padding-oracle attack.
- **Impact:** An attacker on the same network could potentially decrypt
  encrypted traffic between clients and the router admin interface.
- **Reference:** https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566

### Telnet (Port 23) — Open Service Risk
- **Port:** 23/tcp
- **Description:** Telnet transmits all data in plaintext including credentials.
  Anyone on the network capturing traffic with Wireshark can read everything
  sent over Telnet.
- **Recommendation:** Disable Telnet on router. Use SSH instead (currently filtered).

### Expired SSL Certificate
- **Expired:** 2024-12-04 (5+ months ago)
- **Impact:** Browsers and clients cannot verify the router's identity,
  making the admin interface susceptible to impersonation attacks.

---

## Key Takeaways

1. **Attack surface is small** — only 2 live hosts, minimal open ports — good network hygiene overall
2. **Telnet should never be open** on any device — it's a 1969 protocol with zero encryption
3. **SSL POODLE** is a real, confirmed CVE on this router — patching requires a firmware update from the ISP
4. **Linux 3.5 kernel** on the router is significantly outdated (current is 6.x)
5. **Unknown device at .3** — worth investigating (could be a smart TV, IoT device, or guest phone)

---

## Commands Reference

```bash
# Host discovery
sudo nmap -sn 192.168.100.0/24

# Basic port scan
sudo nmap 192.168.100.1

# Service version detection
sudo nmap -sV 192.168.100.1

# OS + aggressive scan
sudo nmap -A 192.168.100.1

# Targeted ports across network
sudo nmap -p 22,80,443,21,23,3306,8080 192.168.100.0/24

# Vulnerability scripts
sudo nmap --script vuln 192.168.100.1
```

## References
- [Nmap Official Documentation](https://nmap.org/docs.html)
- [CVE-2014-3566 — POODLE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566)
- [Nmap NSE Script Reference](https://nmap.org/nsedoc/)
