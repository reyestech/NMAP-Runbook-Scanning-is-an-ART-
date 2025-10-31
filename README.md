# NMAP-Runbook-Scanning-is-an-ART-
<!--
README: Nmap Runbook
Owner: Hector M. Reyes (@reyestech) • SOC Analyst
Tip: To adjust ALL image sizes quickly, search for `data-img-w="90%"` and change it.
-->

<div align="center">
  <img 
    src="https://github.com/user-attachments/assets/80f4773b-4cfa-447c-b74a-7382e3aa8390" 
    alt="Nmap Hero" 
    style="max-width:100%; height:auto;" 
    data-img-w="90%"
    width="90%"
  />
</div>

<h1 align="center">Nmap Runbook</h1>

<p align="center"><b>Hector M. Reyes</b> • SOC Analyst</p>

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/cf4e931c-1885-4f92-8670-e5b0bb24d264" 
    alt="Nmap Runbook Header Graphic" 
    style="max-width:100%; height:auto;" 
    data-img-w="80%"
    width="80%"
  />
</p>

---

## TL;DR

**Flow:** Discover ➜ Enumerate ➜ Fingerprint ➜ (Safe) Vuln Scan ➜ Save Artifacts ➜ Diff Over Time  
**One-liners:**
```bash
# Fast host discovery (noisy but quick)
nmap -sn 10.0.0.0/24 -oA scans/10-0-0-0_discovery

# Top 1000 TCP with service versions + default scripts
nmap -sS -sV -sC -T4 -Pn -oA scans/target_quick 10.0.0.5

# Thorough profile (slow): OS detect + versions + scripts + traceroute
nmap -A -p- -T3 -oA scans/target_full 10.0.0.5

# UDP reconnaissance (top ports)
nmap -sU --top-ports 100 -T3 -oA scans/target_udp 10.0.0.5

# (Safe) vulnerability scripts
nmap --script "vuln and safe" -sV -oA scans/target_vuln 10.0.0.5
```
> [!IMPORTANT] Legal / Ethical: Only scan systems you own or have explicit written permission to test.

---

## Intro (What Nmap Does)
Nmap (Network Mapper) is a multi-platform network scanner used for network discovery, port scanning, service & OS fingerprinting, and scriptable enumeration.
- Network mapping: See what’s alive and how it’s connected.
- Port scanning: Find open/closed/filtered ports (TCP/UDP).
- Service detection: -sV identifies service & version.
- OS detection: -O (best-effort fingerprinting).
- Scripting: NSE automates enumeration & checks (-sC, --script ...).
- Outputs: Text, XML, greppable, “all at once” via -oA.
> [!TIP] Use profiles below to copy/paste scans without remembering every flag.

---

## Install
```bash
# Debian/Ubuntu
sudo apt update && sudo apt install -y nmap

# Fedora
sudo dnf install -y nmap

# Arch
sudo pacman -S nmap

# macOS (Homebrew)
brew install nmap

# Windows (winget)
winget install -e --id Insecure.Nmap
```

---

How to Use This Runbook
- Every section has copyable commands (GitHub’s built-in Copy button appears on fenced code blocks).
- Replace targets like `10.0.0.5` or `10.0.0.0/24` with your scope.
- Store results under `scans/` with `-oA` so you can diff over time.

---

Scan Profiles 
<details> <summary><b>1) Host Discovery (Ping Sweep)</b></summary>
  
```bash
# ICMP + ARP where applicable (fast map of alive hosts)
nmap -sn 10.0.0.0/24 -oA scans/discovery_10-0-0-0_2025-10-31

# If ICMP is blocked, try TCP/ARP discovery
nmap -sn -PS22,80,443 -PA80,443 10.0.0.0/24 -oA scans/discovery_tcp_2025-10-31

```
</details> <details> <summary><b>2) Quick TCP (Top 1000) + Default Scripts + Versions</b></summary>



```bash
nmap -sS -sV -sC -T4 -Pn \
  -oA scans/target_quick_2025-10-31 10.0.0.5
```
</details> <details> <summary><b>3) Full TCP (All Ports) + Thorough Enumeration</b></summary>

```bash
# Slower but comprehensive; includes OS detect & traceroute
nmap -A -p- -T3 -oA scans/target_full_2025-10-31 10.0.0.5
```
</details> <details> <summary><b>4) UDP Top 100 (Recon)</b></summary>
  
```bash
nmap -sU --top-ports 100 -T3 \
  -oA scans/target_udp_top100_2025-10-31 10.0.0.5
```
</details> <details> <summary><b>5) Web App Surface (Common Ports)</b></summary>
  
```bash
# Common web + app ports with versions and basic NSE
nmap -p 80,443,8080,8443,8000,8888,3000,5000,9443 \
  -sS -sV -sC -T4 \
  -oA scans/target_web_2025-10-31 10.0.0.5
```
</details> <details> <summary><b>6) Safe Vulnerability Scripts</b></summary>
  
```bash
# Runs scripts in "vuln" category that are tagged as safe
nmap -sV --script "vuln and safe" \
  -oA scans/target_vuln_safe_2025-10-31 10.0.0.5
```
</details> <details> <summary><b>7) IPv6 Examples</b></summary>
  
```bash
# Neighbor discovery may vary; specify IPv6 addresses explicitly
nmap -6 -sS -sV -T4 -oA scans/ipv6_quick fe80::1234:abcd:5678:9abc
```
</details> <details> <summary><b>8) Using a Target List File</b></summary>

```bash
# targets.txt contains one host or CIDR per line
nmap -sS -sV -sC -iL targets.txt -T4 -oA scans/multi_quick_2025-10-31
```
</details>

---

## Output & Post-Processing
```bash
# Save all formats (normal, XML, greppable) with a single prefix
nmap -oA scans/target_prefix 10.0.0.5

# Quick grep: list open ports from normal output
grep -i "open" scans/target_prefix.nmap | sort -u

# XML to HTML (requires xsltproc)
xsltproc /usr/share/nmap/nmap.xsl scans/target_prefix.xml -o scans/target_prefix.html

# Diff two scans (track change over time)
ndiff scans/old_prefix.xml scans/new_prefix.xml > scans/diff.txt
```
> [!TIP] Use consistent prefixes like scans/host_date_profile to keep artifacts tidy.

---

## NSE (Scripting) Essentials
```bash
# Default script set (good first pass)
nmap -sC -sV 10.0.0.5 -oA scans/default_scripts

# Select by category (auth, discovery, safe, vuln, intrusive)
nmap --script "discovery or default" 10.0.0.5 -oA scans/discovery_pass

# Run specific scripts (example: SSL/TLS checks)
nmap -p 443 --script ssl-cert,ssl-enum-ciphers 10.0.0.5 -oA scans/tls_check
```
> [!NOTE]
Start with safe scripts on production. Reserve intrusive scripts for lab or explicit approval.

---

## Performance & Evasion
```bash
# Timing templates: T0 (paranoid) .. T5 (insane). T4 is a common default for speed.
nmap -sS -T4 10.0.0.5

# Specify ports, speedup with min-rate / max-retries
nmap -p- --min-rate 2000 --max-retries 2 -T4 10.0.0.5 -oA scans/speed_pass

# Firewall evasion (use responsibly)
nmap -sS -f --mtu 16 --data-length 24 --source-port 53 -D RND:5 10.0.0.5 -oA scans/evasion_test
```
> [!CAUTION] Evasion flags can break things or trigger controls. Know your environment and approvals.

---

Troubleshooting
```bash
# Check if a host is really down or just blocking pings
nmap -Pn -sS -p 22,80,443 10.0.0.5 -oA scans/pn_check

# Service detection taking forever? Lower retries / aggressive version intensity
nmap -sV --version-light --max-retries 2 10.0.0.5 -oA scans/sv_light
```

## Appendix: Batch & Diff Workflows
<details> <summary><b>Batch all targets (shell script)</b></summary>

```bash
#!/usr/bin/env bash
# nmap-profiles.sh — quick launcher for common profiles
# Usage: ./nmap-profiles.sh targets.txt scans/  (or pass single IP)

TARGETS="${1:-targets.txt}"
OUTDIR="${2:-scans}"
DATE="$(date +%F)"

mkdir -p "$OUTDIR"

run() { echo ">> $*"; eval "$*"; }

while read -r tgt; do
  [ -z "$tgt" ] && continue
  base="${OUTDIR}/$(echo "$tgt" | tr '/:' '_')_${DATE}"

  run "nmap -sn $tgt -oA ${base}_discovery"
  run "nmap -sS -sV -sC -T4 -Pn $tgt -oA ${base}_quick"
  run "nmap -A -p- -T3 $tgt -oA ${base}_full"
  run "nmap -sU --top-ports 100 -T3 $tgt -oA ${base}_udp100"
  run "nmap -sV --script 'vuln and safe' $tgt -oA ${base}_vuln"
done < <([ -f "$TARGETS" ] && cat "$TARGETS" || echo "$TARGETS")
```
</details> <details> <summary><b>Masscan ➜ Nmap Handoff (very fast discovery)</b></summary>

```bash
# Discover fast with masscan, then hand open ports to Nmap for details
sudo masscan 10.0.0.0/24 -p1-65535 --rate 5000 -oL masscan.lst
grep -i "open" masscan.lst | awk '{print $4}' | sort -u > open_ports.txt
ports="$(paste -sd, open_ports.txt)"
nmap -sS -sV -sC -p "$ports" 10.0.0.0/24 -oA scans/masscan_handoff
```
</details> <details> <summary><b>Schedule & Diff (cron snippet)</b></summary>

```bash
# Discover fast with masscan, then hand open ports to Nmap for details
sudo masscan 10.0.0.0/24 -p1-65535 --rate 5000 -oL masscan.lst
grep -i "open" masscan.lst | awk '{print $4}' | sort -u > open_ports.txt
ports="$(paste -sd, open_ports.txt)"
nmap -sS -sV -sC -p "$ports" 10.0.0.0/24 -oA scans/masscan_handoff
```
</details>


---
---
---
---

# NMAP — Runbook (Scanning is an ART)

**Owner:** Hector M. Reyes (@reyestech) • **Role:** SOC Analyst

> **Ethics:** Only scan systems you own or have **explicit written permission** to test.

---

## TL;DR

**Flow:** Discover → Enumerate → Fingerprint → (Safe) Vuln checks → Save → Diff

```bash
# 1) Fast discovery (ICMP/ARP)
nmap -sn 10.0.0.0/24 -oA scans/discovery

# 2) Quick TCP surface (top 1000) + versions + default scripts
nmap -sS -sV -sC -T4 -Pn 10.0.0.5 -oA scans/quick

# 3) Full TCP (all ports) + OS + traceroute (thorough; slower)
nmap -A -p- -T3 10.0.0.5 -oA scans/full

# 4) UDP recon (top 100)
nmap -sU --top-ports 100 -T3 10.0.0.5 -oA scans/udp100

# 5) (Safe) vuln scripts
nmap -sV --script "vuln and safe" 10.0.0.5 -oA scans/vuln_safe

# 6) Diff two scans over time
ndiff scans/old.xml scans/new.xml > scans/diff.txt
```
##
Install
