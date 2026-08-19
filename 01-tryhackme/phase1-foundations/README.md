# Phase 1 — Foundations

Linux, networking, and core reconnaissance tooling. The base layer everything
else builds on.

## Rooms in this phase

| Room | Status | Notes |
|---|---|---|
| Linux Fundamentals Part 1 | ✅ | Filesystem, navigation, core commands |
| Linux Fundamentals Part 2 | ✅ | File ops, text handling, grep/find |
| Linux Fundamentals Part 3 | ✅ | Permissions, SSH, processes, networking |
| Nmap | ✅ | Worked example — depth standard reference |
| Nmap Live Host Discovery | ✅ | ARP/ICMP/TCP/UDP host discovery |
| Nmap Basic Port Scans | ⬜ | |
| Nmap Advanced Port Scans | ⬜ | |
| Nmap Post Port Scans | ⬜ | |
| Windows Fundamentals 1–3 | ⬜ | |
| What is Networking? | ⬜ | |
| Network Fundamentals | ⬜ | |
| DNS in Detail | ⬜ | |
| HTTP in Detail | ⬜ | |
| Wireshark: The Basics | ⬜ | |
| TShark: The Basics | ⬜ | |

## Depth-upgrade backlog
Your original Linux notes are solid recall references but thin on the "why."
When you revisit, add Layer-2 follow-ups for:
- **Linux Part 3:** What's the security risk of a `777` file? What is SUID and
  why is it a privilege-escalation concern? Why does `/etc/shadow` exist
  separately from `/etc/passwd`? (passwd is world-readable → hashes moved out to
  prevent offline cracking.)
