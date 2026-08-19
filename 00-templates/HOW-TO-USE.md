# 📘 How to Use This Portfolio — Workflow Guide

Your note-taking *system* is what turns "I did 150 labs" into "I can explain and
defend 150 vulnerabilities in an interview." Follow this loop for every room/lab.

---

## The Per-Room / Per-Lab Loop

1. **Solve it first, notes second.** Don't take notes while stuck — finish the
   room, then write up. Notes are for consolidation, not narration.

2. **Copy the right template.**
   - TryHackMe room → copy `00-templates/TEMPLATE-tryhackme-room.md`
   - PortSwigger lab → copy `00-templates/TEMPLATE-portswigger-lab.md`
   - Rename it to the room/lab name, drop it in the correct phase/path folder.

3. **Fill it in your own words.** If you find yourself copy-pasting the room's
   text, stop — rephrase it. The act of rephrasing is the learning.

4. **Force yourself to write Layer 2.** For every interview question, write the
   *follow-up* an interviewer would ask. If you can't, that's your signal to go
   back and understand the "why" before moving on. This is the single most
   valuable habit in the whole system.

5. **Capture 1–2 screenshots** into `_assets/screenshots/`, named clearly
   (e.g. `sqli-union-attack-burp.png`), and reference them in the note.

6. **Write the one-line summary.** If you can't describe what you can now *do*
   in one sentence, you haven't extracted the value yet.

7. **Update the README progress table** and commit to git.

---

## Git Workflow (do this from day one)

```bash
# First time only
cd cybersec-portfolio
git init
git add .
git commit -m "Initial portfolio structure"
# then connect to a GitHub repo and push

# After each room/lab
git add .
git commit -m "Add notes: [Room/Lab name]"
git push
```

**Why commit per room:** the commit history becomes a dated, verifiable record
of consistent work — recruiters can see you've been at this steadily for months,
not crammed in a weekend.

---

## Depth Standard — The Bar to Hit

Compare these two answers to "what does `-sS` do?":

❌ **Shallow (your starting point):**
> "SYN scan. Faster. Doesn't complete the handshake."

✅ **Interview-grade (the bar):**
> "Sends SYN, gets SYN-ACK, replies RST instead of ACK so the handshake never
> completes — which historically meant the app layer didn't log it, hence
> 'stealthy.' It needs root because it crafts raw packets, which is exactly why
> `-sT` exists for unprivileged users."

The facts are the same. The second one gets the offer. Every note should push
toward the second standard — see `01-tryhackme/phase1-foundations/Nmap.md` for a
full worked example.

---

## What Goes Where

| Folder | Put this here |
|---|---|
| `01-tryhackme/phaseN-.../` | One `.md` per room, in its roadmap phase |
| `02-portswigger/<class>/` | One `.md` per lab, grouped by vuln class |
| `03-vdp-engagements/` | Write-ups of real disclosure work (e.g. Servify) |
| `04-projects/` | Standalone tools — your Python vuln scanner, etc. |
| `05-cheatsheets/` | Condensed quick-refs you distil from your own notes |
| `_assets/screenshots/` | All evidence images, clearly named |

---

## Phase 2 Onward — Where It Matters Most

The foundational rooms (Phase 1) just need to be solid. The **SOC-core notes in
Phase 2** (MITRE, Splunk, YARA, Sigma) are the genuinely interview-differentiating
material for the roles you're targeting. Put the most effort into the
"why/when/scenario" depth there — that's where candidates separate themselves.
