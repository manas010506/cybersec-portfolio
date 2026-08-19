# Nmap Live Host Discovery

## Objective

Learn how to identify live hosts on a network using Nmap without performing a full port scan.

---

# What is Host Discovery?

Host discovery is the process of determining which devices on a network are online (alive) before scanning them for open ports and services.

Benefits:
- Reduces scanning time.
- Minimizes unnecessary network traffic.
- Helps identify active systems quickly.

---

# Types of Host Discovery

### ARP Scan

Used on local networks (same subnet).

```bash
nmap -PR 192.168.1.0/24
```

**Key Points**
- Uses ARP Requests.
- Fastest and most reliable method on a local LAN.
- Does not require ICMP.

---

### ICMP Echo Scan (Ping Scan)

```bash
nmap -PE 192.168.1.10
```

**Key Points**
- Sends ICMP Echo Request packets.
- Similar to the `ping` command.
- Some firewalls block ICMP traffic.

---

### ICMP Timestamp Request

```bash
nmap -PP 192.168.1.10
```

Used when ICMP Echo Requests are filtered.

---

### ICMP Address Mask Request

```bash
nmap -PM 192.168.1.10
```

Less commonly used today.

---

### TCP SYN Ping

```bash
nmap -PS22,80,443 192.168.1.10
```

**Key Points**
- Sends TCP SYN packets.
- Useful when ICMP is blocked.
- Checks if a host responds on common TCP ports.

---

### TCP ACK Ping

```bash
nmap -PA80,443 192.168.1.10
```

**Key Points**
- Sends TCP ACK packets.
- Can bypass certain firewall rules.

---

### UDP Ping

```bash
nmap -PU53,161 192.168.1.10
```

**Key Points**
- Sends UDP packets.
- Useful for discovering systems running UDP services.

---

# Scan an Entire Network

```bash
nmap -sn 192.168.1.0/24
```

`-sn` performs host discovery only.

No port scanning is performed.

---

# Disable Host Discovery

```bash
nmap -Pn 192.168.1.10
```

Treats the target as online even if it doesn't respond to ping.

Useful when:
- Firewalls block ICMP.
- Hosts ignore ping requests.

---

# Common Nmap Options

| Option | Description |
|----------|-------------|
| -sn | Host discovery only |
| -Pn | Skip host discovery |
| -PR | ARP Ping |
| -PE | ICMP Echo |
| -PP | ICMP Timestamp |
| -PM | ICMP Address Mask |
| -PS | TCP SYN Ping |
| -PA | TCP ACK Ping |
| -PU | UDP Ping |

---

# Important Commands

```bash
nmap -sn 192.168.1.0/24

nmap -PR 192.168.1.0/24

nmap -PE 192.168.1.10

nmap -PS80,443 192.168.1.10

nmap -PA80 192.168.1.10

nmap -PU53 192.168.1.10

nmap -Pn 192.168.1.10
```

---

# Interview Questions

### What is host discovery?

The process of identifying active devices on a network before performing detailed scans.

---

### Why use `-sn`?

To discover live hosts without scanning ports.

---

### When should you use `-Pn`?

When the target blocks ICMP or other host discovery probes, but you still want to perform a port scan.

---

### Which host discovery method is best on a local network?

ARP Scan (`-PR`), because it's fast and highly reliable on the local subnet.

---

### Why use TCP SYN Ping instead of ICMP Ping?

Some firewalls block ICMP traffic, while TCP packets to common ports may still receive responses.

---

# Cybersecurity Relevance

## SOC Analyst

- Identify active systems during incident response.
- Validate assets before investigation.
- Perform network reconnaissance.

---

## Cloud Security

- Verify active EC2 instances.
- Discover hosts within VPC environments.
- Support cloud asset inventory and network validation.

---

## Penetration Testing

- First step in reconnaissance.
- Helps reduce scan time.
- Identifies valid targets before enumeration.

---

# Key Takeaways

- Host discovery identifies live systems before port scanning.
- ARP Scan is the preferred method on local networks.
- ICMP Echo may be blocked by firewalls.
- TCP and UDP discovery methods help identify hosts when ICMP fails.
- `-Pn` forces Nmap to assume the host is online.

---

# Skills Acquired

- Network reconnaissance
- Host discovery techniques
- Nmap host discovery options
- Basic network enumeration
- Scan optimization