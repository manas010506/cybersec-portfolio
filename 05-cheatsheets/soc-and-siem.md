1. SOC tiers — who does what, and that false positives feed back into rule tuning

Tier 1 — Alert Triage
Monitors incoming alerts.
Determines whether an alert is likely benign, suspicious, or a true incident.
Performs initial investigation and enrichment.
Escalates meaningful alerts to Tier 2.
Documents false positives and why they occurred.

Tier 2 — Incident Investigation
Takes escalated alerts.
Performs deeper investigation and correlation.
Determines scope, impact, and attack technique.
Coordinates containment and remediation.
Escalates sophisticated cases to Tier 3.

Tier 3 — Threat Hunting / Advanced Analysis
Investigates advanced or previously unknown threats.
Performs proactive threat hunting.
Conducts malware/forensic analysis when required.
Develops or improves detection strategies.
Identifies gaps in existing security controls.
The feedback loop

A mature SOC isn't simply:

Alert → Analyst → Close

It's:

Detection → Tier 1 → Tier 2 → Tier 3 → Detection Engineering → Improved Detection

And importantly:

False Positive → Why was it false? → Tune the detection rule → Fewer unnecessary alerts

For example, if a SIEM rule generates an alert every time a particular service account logs in from a new IP, Tier 1 might discover that the account legitimately does this every night. Rather than repeatedly closing the alert, the SOC can tune the rule using that knowledge.

However, false-positive feedback isn't necessarily only from Tier 1. Tier 2/3 investigations can also reveal that a detection is too noisy, too broad, or missing important context.

So the broader principle is:

SOC analysts don't just consume detections; their investigation outcomes continuously improve the detection system.

This is one of the key differences between a mature SOC and a basic alert-monitoring operation.


2. Triage questions — the concrete ones: expected for this user/host/time? what else did the host do nearby? has it fired before? source and destination reputation?

“I usually start by understanding whether the activity is expected or unusual. I look at the user, host, and time of the event and ask whether this is normal behaviour for them.

Then I check the surrounding activity — what happened before and after the alert, and whether there are other events that could indicate a larger attack.

I also check whether the same user or host has triggered similar alerts before, because a repeated pattern can provide useful context.

Finally, I investigate the source and destination, including IP or domain reputation, and check whether the communication is legitimate.

Based on all that context, I decide whether it's a false positive, a suspicious event that needs monitoring, or a genuine incident that should be escalated.”


3. Aggregation → normalisation → correlation, with the src_ip / IpAddress / sourceIPAddress example

1. Aggregation — Bring it together

The SIEM collects logs from:

Firewall + Endpoint + Cloud + Authentication + IDS → SIEM

At this stage, they're simply being stored/ingested centrally.

2. Normalisation — Make the fields consistent

The SIEM understands that:

src_ip
IpAddress
sourceIPAddress

all represent the same concept.

It can therefore map them to a common field:

source_ip = 185.10.20.5

Now queries and detection rules don't have to separately understand every vendor's naming convention.

3. Correlation — Connect the events

Now suppose the SIEM sees:

10:01  → Firewall: connection from 185.10.20.5
10:02  → VPN: successful login from 185.10.20.5
10:04  → Endpoint: suspicious PowerShell execution
10:05  → Cloud: unusual data access from 185.10.20.5

Because the events have been normalised, the SIEM can correlate them using the common source_ip.

Instead of treating these as four unrelated alerts, it can recognise:

“Multiple suspicious activities are associated with the same source IP within a short time window.”

That gives the analyst a much stronger signal to investigate.

4. Why normalisation must precede correlation

“I think of it as collect, standardise, and connect.

Aggregation means collecting logs from different sources such as firewalls, endpoints, authentication systems, cloud services, and IDS into a central SIEM.

Normalisation means converting those different log formats and field names into a common structure. For example, one firewall might call the source IP src_ip, Windows might use IpAddress, and a cloud service might use sourceIPAddress. The SIEM can map all of these to a common field such as source_ip.

This needs to happen before correlation, because correlation relies on being able to recognise that different events are referring to the same thing. Without normalisation, a correlation rule may not realise that src_ip, IpAddress, and sourceIPAddress are actually the same field.

Once the data is normalised, correlation can connect related events across different sources and identify a larger pattern or potential incident.”


5. What makes an alert good — actionable, low false-positive rate, tells you what to do next

“A good security alert should be actionable, reliable, and provide enough context for the analyst to know what to do next. It should have a relatively low false-positive rate, because if analysts constantly receive alerts that turn out to be legitimate activity, they may eventually start ignoring them. The alert should also contain useful context such as the affected user or host, source and destination IP, timestamp, severity, and related events. Most importantly, it should explain why the activity is suspicious and what the analyst should investigate or do next, such as verifying the user, checking related activity, isolating a host, blocking an IP, or escalating the incident. So, in simple terms, a good alert tells you what happened, why it matters, and what to do next.”