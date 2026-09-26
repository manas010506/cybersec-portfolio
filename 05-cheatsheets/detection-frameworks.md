Both done. Questions before notes — and these two are conceptual, so it's the reasoning I want, not definitions.

**Pyramid of Pain**

--

1. Name the six levels, bottom to top.
a. Hash Values
b. IP Addresses
c. Domain Names
d. Network Artifacts
e. Tools
f. Tactics, Techniques and Procedures

--

2. Why is detecting on a file hash nearly worthless, and what specifically does detecting on TTPs cost an attacker?

Detecting threats using a file hash alone is limited because hashes are tied to a specific file. Even a small modification to the malware can produce a completely different hash, allowing a modified sample to evade the detection.

Hash-based detection is therefore useful for **known, unchanged malware**, but is less effective against modified or previously unseen samples.

TTP-based detection focuses on **how an attacker operates** rather than the exact file. It can detect behaviors such as suspicious PowerShell execution, credential dumping, or unusual remote access. This increases the cost for the attacker because changing the malware is easy, but changing the underlying technique or attack workflow may require significant changes to their tools, infrastructure, or objectives.

--

3. You're writing a CloudTrail detection. Where on the pyramid does "alert on this specific IP address" sit, and how would you move that detection higher?

“Alerting on a specific IP address” sits at the **bottom of the Pyramid of Pain**, because an attacker can often change or rotate their IP address relatively easily.

To move the detection higher, I would detect **behavior or TTPs** instead of a single IP. For example, I could alert when AWS credentials associated with an EC2 instance are used from an unexpected source, such as outside the organization's known network ranges or from an unusual location.

This is harder for an attacker to evade because changing the IP alone would not necessarily bypass the behavioral detection.

--

**MITRE**

--

4. Tactic vs technique vs procedure — what's the difference, and why does the distinction matter when you're writing a detection?

**Tactics** represent the attacker's high-level goal or **“why”**, such as achieving persistence or gaining credential access.

**Techniques** describe the general method or **“how”** used to achieve that goal, such as using PowerShell for execution.

**Procedures** describe the specific implementation or **“exactly how”** an attacker carries out the technique, such as a particular PowerShell command or script.

The distinction matters when writing detections because the level determines how resilient the detection is. A **procedure-level detection** may catch one specific implementation but can easily break when the attacker changes their command or tool. A **technique-level detection** focuses on the underlying behavior, so it can catch different implementations of the same technique. Tactics are generally too broad to directly alert on.

**In practice, technique-level detection is often the useful middle ground:** specific enough to detect meaningful behavior but broad enough to remain effective when attackers change their implementation.

--

5. Your Splunk investigation had account creation → WMIC remote execution → encoded PowerShell. Map those to tactics, and name the technique IDs you can recall.

Account creation → Persistence — T1136, Create Account
WMIC remote execution → Execution / Lateral Movement — T1047, Windows Management Instrumentation
Encoded PowerShell → Execution — T1059.001, PowerShell, with T1027 Obfuscated Files or Information

6. What does ATT&CK give a SOC that a list of IOCs doesn't?

--

MITRE ATT&CK gives a SOC behavioral context and a framework for detection, while a list of IOCs mainly gives specific things to block or search for.

IOCs → What did the attacker leave behind?
Examples: IP addresses, domains, file hashes.
ATT&CK → How did the attacker operate?
Examples: PowerShell execution, credential dumping, scheduled tasks, remote services.
Why it matters

An IOC can become useless when the attacker changes their IP, domain, or file hash. An ATT&CK technique describes the underlying behavior, so a SOC can build detections that identify different implementations of the same attack method.

--