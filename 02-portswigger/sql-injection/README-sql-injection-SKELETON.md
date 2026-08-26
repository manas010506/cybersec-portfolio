# SQL Injection — PortSwigger Path

> **Labs completed:** 10 / 18 · **Path status:** 🔄 In progress
> **Burp edition:** Community · **Last updated:** 2026-08-26

---

## 1. What SQL Injection Actually Is
<!-- PROMPT: Explain it to a non-technical hiring manager in 3-4 sentences.
Do not use the words "sanitise", "malicious input", or "hacker". The clearest
version usually explains that the application mixes DATA and CODE in the same
string, and the database cannot tell which is which. -->

SQLi is a type of an attack in which an attacker is able to modify, delete or view the data which is to accessible to attacker. It is designed in such a way that the database is confused and cannot tell whether the attack has come from the developer or the application user

---

## 2. How I Find It — Methodology
<!-- PROMPT: You've done ten labs. Reconstruct the process you actually used,
in order. Be honest about what came from the lab title vs what you'd do blind. -->

1. **Where I look:**
a. URL path and query parameters
b. HTTP method (GET, POST, etc.)
c. Request body/form fields
d. HTTP headers

2. **What I send first:**
I first send a normal, controlled request and record the baseline response.
Then I change one input at a time with a harmless variation—such as changing its type, length, value, or format—rather than immediately sending an exploit payload.
The reason is that a baseline tells me whether a later difference is actually caused by my input rather than normal application behavior.
3. **What tells me I'm onto something:** <!-- three distinct signals -->
4. **How I confirm:** <!-- the test that rules out coincidence -->

---

## 3. Exploitation — The Progression
| Stage                             | What I send                                                                                                | What it proves                                                                      |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Detect**                        | A normal request, followed by a harmless input variation that changes the query's expected syntax/behavior | The input is influencing backend query processing                                   |
| **Confirm injectable**            | A controlled pair of inputs that should produce predictably different database behavior                    | The observed behavior is actually caused by injection, not normal application logic |
| **Determine column count**        | A sequence of `ORDER BY` column-index tests or an equivalent `UNION` column-count test                     | How many columns the original query returns                                         |
| **Find a text-compatible column** | A harmless `UNION SELECT` test using a recognizable text marker in each candidate position                 | Which returned column can accept/display text                                       |
| **Identify the DBMS**             | A DBMS-specific, non-destructive fingerprint/version check appropriate to the lab                          | Which database engine is processing the query                                       |
| **Enumerate schema**              | Read-only metadata/catalog queries to identify databases, tables, and columns                              | The structure of the database                                                       |
| **Extract data**                  | A targeted read-only query against the identified table/column(s)                                          | The actual records stored in the target fields                                      |

Why column count must come before a UNION attack

A UNION combines the result of two SELECT statements. The two queries must return the same number of columns (and the corresponding columns must have compatible types).

So before a UNION SELECT can reliably work, I need to know the number of columns returned by the original query. Otherwise, the database will typically reject the combined query because the result sets don't have matching structures.

## 4. The Hard Cases

### Blind SQL injection with conditional responses

* **What changed when the output disappeared?**
  I stopped looking for the database output itself and instead looked for a **side-channel difference** in the application's response. The page effectively became a yes/no indicator.

* **How did I extract the string?**
  I converted each unknown character into a series of Boolean questions: first identify the character position, then test conditions about its value. Each response narrowed the possible characters until the complete string could be reconstructed. In practice, binary-search-style comparisons can make this more efficient than testing every character individually.

* **Roughly how many requests? What does that mean for SOC?**
  It can take **dozens to hundreds of requests**, depending on the string length and extraction method. The SOC implication is important: a blind attack may not produce one spectacular malicious request. Instead, it can generate a **repetitive sequence of nearly identical requests with systematically changing parameters**, which can be detected through request-frequency, parameter-variation, and response-correlation analysis.

---

### Determining the number of columns

**`ORDER BY` and `UNION SELECT NULL` answer related but slightly different questions.**

* **`ORDER BY`** tests whether a particular column position exists. When the position becomes invalid, the behavior changes, allowing the column count to be inferred.
* **`UNION SELECT NULL`** tests whether a UNION result has the correct structure. Increasing the number of `NULL` values until the UNION is accepted identifies the required number of columns without initially worrying about data types.

I'd prefer **`ORDER BY`** when I simply need the original query's column count. I'd prefer **`UNION SELECT NULL`** when I'm specifically moving toward a UNION-based test and also want to establish whether the result structure is compatible.

---

### DBMS fingerprinting: Oracle / MySQL / MSSQL

One syntax difference that stood out was that **Oracle requires a `FROM` clause for a normal `SELECT`**, even when I'm selecting a constant rather than reading from an actual table.

Oracle uses:

```sql
SELECT 'test' FROM dual
```

while MySQL can use:

```sql
SELECT 'test'
```

without `FROM`.

**Why `DUAL`?**
`DUAL` is Oracle's special one-row table. It provides the required `FROM` source when the query doesn't need to retrieve data from a real table. This syntax difference is therefore useful when distinguishing Oracle from other DBMSs during fingerprinting.


---

## 5. Impact
The SQL injection vulnerability allows an unauthenticated attacker to enumerate the application's database structure and potentially access sensitive records beyond the intended application interface, creating a risk of unauthorized disclosure of customer, account, or other confidential information.

---

## 6. Remediation
### The actual fix

**Use parameterized queries / prepared statements** as the primary fix.

Instead of concatenating user input into SQL:

```sql
SELECT * FROM users WHERE username = 'INPUT'
```

the application sends the SQL structure separately from the value:

```sql
SELECT * FROM users WHERE username = ?
```

and supplies the user's input as a **parameter**.

**Why this stops SQL injection:** the database parses the SQL statement structure before treating the supplied value as data. Therefore, SQL syntax contained inside the parameter is not interpreted as part of the query. In other words, an attacker can control the **value**, but cannot turn that value into new SQL operators, clauses, or commands.

---

### Defence in depth

Ranked by importance:

1. **Parameterized queries — primary defence**
   Prevents untrusted input from changing SQL syntax. This should be the application's fundamental fix.

2. **Least-privilege database account — damage limitation**
   The application's DB account should have only the permissions it actually needs. If SQL injection occurs, the attacker inherits those limited permissions rather than unrestricted database access.

3. **WAF — additional detection/blocking layer**
   A Web Application Firewall can recognize and block some common SQLi patterns, but it is not a substitute for fixing the application. Attackers can sometimes bypass signature-based rules.

4. **Error suppression / safe error handling — information reduction**
   Don't expose database errors, SQL statements, table names, or stack traces to users. This makes reconnaissance harder but **does not prevent SQL injection**.

So: **prevent → limit impact → detect/block → reduce information leakage.**

---

### What does NOT work reliably

* **Blacklisting quotes or SQL keywords:**
  Attackers can often use alternative syntax, encoding, case variations, comments, or other database-specific behavior. SQL has too many valid representations for a blacklist to reliably define “everything malicious.”

* **Escaping input:**
  Escaping can work only when implemented perfectly for the **specific database, encoding, SQL context, and API**. Mistakes in escaping or character-set handling can reintroduce injection. It is therefore inferior to parameterization.

* **Stored procedures used carelessly:**
  Stored procedures are **not automatically safe**. A procedure that constructs SQL dynamically by concatenating an input value can still be injectable. The procedure itself must use parameterized/dynamic SQL safely.

**Bottom line:** the real remediation is not “filter the attacker's payload”; it is to **prevent user-controlled data from becoming SQL syntax in the first place.**

---

## 7. Detection — Blue Team View
### UNION-based SQL injection in web-server logs

A UNION-based attack can appear as a sequence of requests where an otherwise normal parameter contains **SQL-specific syntax or unusual query-manipulation patterns**. In access logs, I would look for:

* Repeated requests to the same endpoint with systematically modified parameters.
* SQL-related keywords/operators appearing where ordinary application values are expected.
* Abnormally long or encoded parameter values.
* A pattern of requests testing different query structures rather than normal user behavior.

The important SOC signal isn't necessarily one suspicious request; it's the **sequence and progression of requests from the same client/session**.

### Blind SQL injection in logs

A blind attack is harder to spot because the attacker does not need the application to return database contents. The requests can therefore look much closer to legitimate traffic.

A typical pattern is:

> Same endpoint → same parameter → tiny input modification → response comparison → repeat hundreds of times.

The suspicious behavior is the **systematic iteration**, not necessarily an obvious SQL keyword. An attacker can extract information through small Boolean conditions or timing differences while the visible page remains almost identical.

### What I would alert on

I'd use a combination of signals rather than a single keyword rule:

1. **Repeated parameter mutation** — many requests to the same endpoint where one parameter changes systematically.
2. **SQLi indicators** — suspicious SQL syntax, encoding patterns, comments, or database-specific constructs in unexpected parameters.
3. **Response correlation** — repeated requests that produce unusual binary/timing differences.
4. **High request volume** — particularly when concentrated on one parameter/session.
5. **Anomalous client behavior** — a user/session suddenly performing hundreds of structured tests against an endpoint they normally access only occasionally.

A stronger detection would correlate **request content + frequency + parameter variation + response behavior**.

### What benign traffic could trigger it?

There are several false-positive sources:

* Security scanners and vulnerability-management tools.
* QA/automated testing frameworks.
* Developers testing malformed input.
* API clients that legitimately send encoded or unusual strings.
* Search fields containing words such as `UNION`, `SELECT`, or punctuation that resembles SQL.
* Applications that legitimately generate many similar requests.

So I would **not alert simply because a request contains `UNION` or `'`**. I'd use contextual scoring and establish a baseline for the application. A single suspicious request may be noise; **hundreds of systematically changing requests against the same parameter are much more meaningful.**


---

## 8. Burp Workflow
## 8. Burp Workflow

| Burp tool                | How I used it                                                                                                                                                                                 |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxy / HTTP history** | Captured the application's requests and responses, then inspected parameters, headers, cookies, methods, and response differences to identify interesting inputs.                             |
| **Repeater**             | Sent the same request repeatedly while changing one parameter at a time. I used it to confirm suspected behavior and compare baseline vs. modified responses.                                 |
| **Intruder**             | Automated controlled, repetitive testing of a parameter when manually trying many values would be inefficient. It helped identify response differences and patterns across multiple requests. |
| **Decoder**              | Decoded URL/Base64 and other encoded values so I could understand what was actually being sent, and encoded values when needed for controlled testing.                                        |


---

## 9. Interview Q&A — Two Layers
1. What is SQL injection?
Follow-up: “Why does parameterized SQL actually prevent it? Explain what happens between the application sending the query and the database executing it.”

2. How do you exploit SQL injection when the application returns no data?
Follow-up: “If you only get a true/false response, how can you determine an unknown value, and what pattern would you expect to see in the server logs?”

3. How would you fix SQL injection?
Follow-up: “If the developer says, ‘We already escape quotes and have a WAF, so we're protected,’ how would you explain why that isn't a sufficient fix?”

---

## 10. Labs Completed

| # | Lab | Level | Notes |
|---|---|---|---|
| 1 | SQL injection in WHERE clause allowing retrieval of hidden data | Apprentice | |
| 2 | SQL injection allowing login bypass | Apprentice | |
| 3 | Querying the database type and version on Oracle | Practitioner | |
| 4 | Querying the database type and version on MySQL and Microsoft | Practitioner | |
| 5 | Listing the database contents on non-Oracle databases | Practitioner | |
| 6 | UNION attack, determining the number of columns | Practitioner | |
| 7 | UNION attack, finding a column containing text | Practitioner | |
| 8 | UNION attack, retrieving data from other tables | Practitioner | |
| 9 | UNION attack, retrieving multiple values in a single column | Practitioner | |
| 10 | Blind SQL injection with conditional responses | Practitioner | |

**Remaining:** visible error-based · blind with conditional errors · blind with
time delays · blind with time delays and information retrieval · filter bypass
via XML encoding · *(2 out-of-band labs require Burp Pro — N/A)*

---

## 11. One-Line Summary
I can systematically identify, validate, and assess SQL injection vulnerabilities, extract information through controlled blind techniques, and recommend effective application and SOC-level defenses.
