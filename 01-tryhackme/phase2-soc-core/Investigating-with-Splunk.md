# Investigating with Splunk

> **Platform:** TryHackMe · **Phase:** 2 (SOC Core) · **Difficulty:** Medium
> **Date completed:** 2026-09-26 · **Time spent:** [2h]
> **Status:** ✅ Complete

---

## 1. Objective
Use Splunk to investigate a simulated intrusion end to end — starting from a
single suspicious indicator and pivoting across event sources to reconstruct the
full attack chain. This is the first room on the roadmap that mirrors what a SOC
analyst actually does day to day: not running a tool, but following evidence.

---

## 2. Core Concepts

### Index vs sourcetype
- **Index** — the physical repository where Splunk stores ingested data. A
  container. `index="main"` scopes a search to one of them.
- **Sourcetype** — a declaration of *what format the incoming data is in*, which
  tells Splunk how to parse it and which field extractions to apply.

The important consequence: **sourcetype is where normalisation happens.** If the
sourcetype is wrong or missing, fields don't extract cleanly — and if fields
don't extract, you can't correlate across sources, because the correlation
engine has no matching field names to join on. A firewall's `src_ip` and
CloudTrail's `sourceIPAddress` only become comparable once parsing has mapped
them into a common schema.

### The pipe
SPL chains commands with `|`. Each command's **results** become the next
command's input — it's passing events, not passing commands. `index="main" |
stats count` retrieves events from the index, then hands that result set to
`stats`, which counts them.

### Time range is a performance lever, not a filter
Splunk is time-indexed, so the time range determines **how many events get
scanned in the first place**. Widening from 24 hours to 30 days doesn't add
commands — it reads thirty times the data through the same pipeline. This is the
single biggest control on search performance, which is why it's set before the
search rather than adjusted afterwards.

### What `stats` changes
Raw search results answer *what happened* — individual events, read one at a
time. `stats` answers *what stands out*. `stats count by src_ip` across a
million authentication events collapses them into a table where one address has
4,000 failures and every other has three. That outlier is invisible when reading
events sequentially, and finding it is most of what detection work is.

---

## 3. Commands & Tools
| Command                                  | What it does                                             | When to use it                                                              |
| ---------------------------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------- |
| `index="main"`                           | Searches all events in the `main` index                  | Starting point to understand the available dataset                          |
| `index="main" EventID=4720`              | Finds user-account creation events                       | Hunting for suspicious account creation/persistence                         |
| `index="main" EventID=13 A1berto`        | Searches Sysmon registry events related to `A1berto`     | Investigating registry activity associated with a suspicious account        |
| `index="main" EventID=1 A1berto`         | Finds Sysmon process-creation events involving `A1berto` | Investigating commands/processes used by or related to a suspicious account |
| `index="main" EventID=4688 A1berto`      | Searches Windows process-creation events                 | Investigating process execution and command-line activity                   |
| `index="main" EventID=4624 A1berto`      | Searches successful Windows logons for the account       | Checking whether the suspicious account was actually used to log in         |
| `index="main" EventID=4625 A1berto`      | Searches failed Windows logons                           | Looking for attempted authentication using the suspicious account           |
| `index="main" A1berto`                   | Searches all events containing the username              | Broad pivot after discovering a suspicious account                          |
| `index="main" powershell`                | Searches for PowerShell-related activity                 | Investigating possible script execution or PowerShell-based attacks         |
| `index="main" EventID=1 powershell`      | Searches process-creation events involving PowerShell    | Identifying suspicious PowerShell execution                                 |
| `index="main" \| stats count by EventID` | Counts events grouped by Event ID                        | Understanding the types and volume of events in the dataset                 |
| `index="main" \| stats count by host`    | Counts events for each host                              | Identifying hosts with unusually high/low event activity                    |


---

## 4. Walkthrough Notes
index="main"
      ↓
Started with the complete dataset
      ↓
EventID=4720
      ↓
Found a newly created suspicious account: A1berto
      ↓
EventID=13 / Registry
      ↓
Investigated registry activity related to A1berto
      ↓
EventID=1 / 4688
      ↓
Investigated process creation and command execution
      ↓
4624 / 4625
      ↓
Checked successful and failed login attempts
      ↓
PowerShell activity
      ↓
Investigated suspicious PowerShell execution and encoded activity
---

## 5. Interview Q&A — Two Layers
Q: Walk me through an investigation you've done.

A:

I worked through the TryHackMe Investigating with Splunk scenario. I started with index="main" and set the time range to All Time because the lab had an unknown scope. I then searched for Event ID 4720, which showed a newly created account called A1berto. From there, I pivoted through registry activity, process creation, login events, and PowerShell activity to understand what the account was associated with. The investigation eventually showed a chain of suspicious account creation, remote execution, and PowerShell activity.

↳ Likely follow-up: You found one suspicious account. What was your next search, and why that one?

A:

I used the account name as my pivot and searched for related registry and process activity. For example, I used EventID=13 with A1berto to investigate registry activity, followed by process-creation events. The reason was to determine what the newly created account was being used for and what activity was associated with it, rather than treating the account creation as an isolated event.

Q: What is a sourcetype in Splunk?

A:

A sourcetype identifies the type and format of data coming into Splunk. It helps Splunk understand how to parse and categorize the events. For example, WinEventLog:Security identifies Windows Security Event Log data.

↳ Likely follow-up: Why does getting the sourcetype wrong break correlation?

A:

If the sourcetype is wrong, Splunk may parse fields incorrectly or fail to extract important fields. That makes it harder to search and correlate events across logs. For example, if the user, host, or EventID fields aren't extracted correctly, I may not be able to connect a login event to the suspicious account or host I'm investigating.

---

## 6. Security Relevance — By Role

- **SOC Analyst:** this is the core loop of the job — take one indicator,
  establish whether it's real, then pivot across data sources until you have a
  chain you can hand to L2 or close with confidence. The specific artifacts here
  (account creation events, process creation, registry writes, PowerShell
  execution) are the highest-frequency telemetry in Windows environments.
- **Cloud Security:** the pivot logic transfers directly. In CloudTrail the
  indicator is an IAM principal or access key rather than a username, but the
  method is identical — one confirmed artifact becomes the search key that pulls
  every related event across services into a single timeline. Same skill,
  different log source.
- **AppSec / VAPT:** shows the defender's side of the attacks you practise
  offensively — useful for writing remediation advice that accounts for what
  will actually be detected and what won't.

---

## 7. Gotchas & Things I Got Wrong
- Searched **All Time**, which was correct in a lab with unknown scope but is
  the wrong instinct in production — it scans everything and is slow for
  everyone. In a real SOC, anchor to the alert timestamp and expand the window
  outward only as needed.
- Initially thought of the pipe as passing commands along. It passes **results**
  — each command operates on the output of the one before it.
- Initially treated index and sourcetype as interchangeable. They are different: the index tells Splunk where the data is stored, while the           sourcetype identifies what kind of data it is.
- At first, I focused on individual events instead of looking at the bigger pattern. Using commands like stats can summarize large amounts of data and make unusual activity easier to spot.
- When I found A1berto, I had to pivot using that artifact rather than continuing with broad searches. This helped connect the account creation to registry, process, login, and PowerShell activity.
- Event IDs only make sense in context. Finding an event such as 4720 doesn't automatically mean an attack; I needed to investigate the surrounding activity and correlate it with other events

---

## 9. One-Line Summary (for resume/LinkedIn)

Investigated a multi-stage intrusion in Splunk by analyzing Windows logs, pivoting across Event IDs, and correlating account creation, process execution, registry, and PowerShell activity to reconstruct the attack chain.

---

## TODO after MITRE
<!-- Come back once the MITRE room is done and map the chain to ATT&CK technique
IDs: account creation (persistence), WMIC remote execution (T1047), encoded
PowerShell (T1059.001 + obfuscation). That mapping is what turns this from a
walkthrough into a detection-engineering artifact. -->