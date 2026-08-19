# Nmap

> **Platform:** TryHackMe · **Phase:** 1 (Foundations) · **Difficulty:** Easy–Med
> **Date completed:** 2026-XX-XX · **Time spent:** ~2h
> **Status:** ✅ Complete
>
> *This is a worked EXAMPLE showing the depth to aim for. Compare it to your
> original Nmap notes — same facts, but with the "why" and "follow-up" layers
> that interviews actually test. Use this as the bar for Phase 2 onward.*

---

## 1. Objective
Use Nmap for host discovery, port scanning, service/OS detection, and basic
recon — the first tool in nearly every engagement and a daily tool for SOC,
cloud, and pentest work.

---

## 2. Core Concepts

### TCP Three-Way Handshake (why scan types exist)
A normal TCP connection = SYN → SYN-ACK → ACK. Nmap's scan types differ in
*how much of this handshake they complete*, and that single fact explains the
stealth/speed/reliability tradeoffs between them. Understand the handshake and
every scan flag stops being something to memorise and becomes something you can
reason about.

### Port States — what each really means
- **Open** — a service is actively listening. The thing you're hunting for.
- **Closed** — host is up and reachable, but nothing is listening on that port.
- **Filtered** — a firewall/filter dropped the probe; Nmap got no clear reply.
  *Filtered ≠ closed* — it usually means something is deliberately blocking you.
- **Unfiltered** — reachable but Nmap can't tell open vs closed (seen in ACK scans).
- **open|filtered** / **closed|filtered** — Nmap genuinely couldn't decide,
  common in UDP scanning because of how UDP responds (see gotchas).

---

## 3. Commands & Tools

| Command | What it does | When to use it |
|---|---|---|
| `nmap -sn <range>` | Host discovery only, no port scan | First pass — find live hosts before wasting time scanning dead IPs |
| `nmap -sS <target>` | SYN "stealth" scan | Default for pentests; fast, needs root/raw sockets |
| `nmap -sT <target>` | Full TCP connect scan | When you DON'T have raw-socket privileges (unprivileged user) |
| `nmap -sU <target>` | UDP scan | Hunting DNS/DHCP/SNMP; slow — scope it to specific ports |
| `nmap -sV <target>` | Service + version detection | Once ports are known, ID the actual software + version |
| `nmap -O <target>` | OS fingerprinting | Building target profile; needs at least one open + one closed port |
| `nmap -A <target>` | Aggressive: -O, -sV, default scripts, traceroute | Quick deep look when stealth doesn't matter |
| `nmap -p- <target>` | All 65,535 TCP ports | When you suspect services on non-standard ports |
| `nmap -sC <target>` | Run default NSE scripts | Light enumeration after finding open ports |
| `nmap -oA <name>` | Save output in all 3 formats | ALWAYS — documentation for reports and later grep |

---

## 4. Walkthrough Notes
- Started every box with `-sn` on the subnet to map live hosts, then `-p-` only
  on the ones that responded. Scanning a /24 with full port range up front is
  the rookie time-sink.
- `-sV` repeatedly turned a boring "port 8080 open" into "Apache Tomcat 9.0.x" —
  which is the line that actually leads somewhere (known CVEs for that version).

---

## 5. Interview Q&A — Two Layers

### Q: What's the difference between `-sS` and `-sT`?
**A:** `-sS` (SYN scan) sends a SYN, waits for SYN-ACK, then sends RST instead
of completing the handshake. `-sT` (connect scan) completes the full three-way
handshake using the OS's networking stack.

> **↳ Likely follow-up: So why is `-sS` considered "stealthier"?**
> **A:** Because the connection is never fully established, many applications
> never log it — historically the app only sees a completed connection, so a
> half-open SYN slips past application-level logging. It's also why `-sS` needs
> raw-socket (root) privileges: it's crafting packets directly instead of asking
> the OS to open a normal socket. That privilege requirement is exactly why
> `-sT` exists as the fallback for unprivileged users. *(Note: modern firewalls
> and IDS detect SYN scans easily, so "stealth" is relative — interviewers like
> when you add that nuance.)*

### Q: Why is UDP scanning (`-sU`) so slow and unreliable?
**A:** UDP is connectionless — there's no handshake. An open UDP port often
sends *no reply at all*, so Nmap can't distinguish "open" from "packet got
dropped." It relies on ICMP "port unreachable" messages to confirm closed ports,
and those are rate-limited by most OSes.

> **↳ Likely follow-up: So how do you make a UDP scan actually usable?**
> **A:** Don't scan all UDP ports — scope it to the ones that matter (53, 67,
> 123, 161). Combine with `-sV` so version probes coax a response out of open
> ports that would otherwise stay silent. Accept that UDP results are lower
> confidence than TCP and confirm anything interesting manually.

### Q: What does "filtered" tell you that "closed" doesn't?
**A:** Closed means the host actively replied "nothing here" (RST). Filtered
means *something dropped your probe* with no reply — almost always a firewall.

> **↳ Likely follow-up: From a recon standpoint, why do you care about that distinction?**
> **A:** Closed ports tell me the host is up and the firewall isn't interfering —
> useful for OS detection. A wall of "filtered" tells me there's a filtering
> device in the path, which itself is intelligence: it shapes whether I try
> `-Pn`, switch to ACK probes to map firewall rules, or change my approach
> entirely.

---

## 6. Security Relevance — By Role
- **SOC Analyst:** validate suspicious hosts, confirm what services are actually
  exposed during an incident, and — crucially — recognise what an Nmap scan
  looks like *in your own logs* when an attacker runs one against you.
- **Cloud Security:** audit EC2 instances against their Security Group rules —
  catch the gap between "what the SG says is open" and "what's actually
  reachable."
- **AppSec / VAPT:** first recon step — map the attack surface before any
  web testing begins.

---

## 7. Gotchas & Things I Got Wrong
- Wrote port states as if "filtered = closed." They're not — filtered means a
  firewall is involved, which is its own signal.
- Tried `-O` on a host with no closed ports and got unreliable results — OS
  detection needs at least one open AND one closed port to fingerprint well.
- Forgot `-oA` on an early box and had to re-run the whole scan to document it.
  Now it's muscle memory on every scan.

---

## 8. One-Line Summary (for resume/LinkedIn)
> Performed network reconnaissance and service enumeration with Nmap — host
> discovery, port-state analysis, and version/OS fingerprinting — and can reason
> about scan-type tradeoffs (SYN vs connect vs UDP) rather than just running flags.
