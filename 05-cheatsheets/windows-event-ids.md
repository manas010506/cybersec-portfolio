# Windows Event IDs — Quick Reference

> Personal cheatsheet. Not a portfolio artifact — written to be re-read.

## Authentication & Logon

| ID       | Meaning                                  | What I'd check next                                                  |
| -------- | ---------------------------------------- | -------------------------------------------------------------------- |
| **4624** | Successful logon                         | Check **Logon Type, username, source IP, and host**                  |
| **4625** | Failed logon                             | Check **source IP, username, Logon Type, and repeated attempts**     |
| **4634** | Logoff                                   | Correlate with the **corresponding 4624** and session duration       |
| **4647** | User-initiated logoff                    | Check **which user/host** and what happened during the session       |
| **4672** | Special privileges assigned to new logon | Check **account, Logon ID, and what activity followed**              |
| **4768** | Kerberos TGT requested                   | Check **account, source IP, and unusual authentication patterns**    |
| **4769** | Kerberos service ticket requested        | Check **target service, account, source host, and unusual requests** |
| **4771** | Kerberos pre-authentication failed       | Check **source IP, account, and repeated failures**                  |


### Logon Types (the field that carries the signal)

|| Type  | Meaning                   | Notes                                                                                                 |
| ------ | ------------------------- | ----------------------------------------------------------------------------------------------------- |
| **2**  | Local interactive         | User logged in directly at the machine                                                                |
| **3**  | Network                   | Accessed the computer over the network, e.g. SMB/file share                                           |
| **4**  | Batch                     | Used by a batch job or scheduled process                                                              |
| **5**  | Service                   | Windows service logged on using an account                                                            |
| **7**  | Unlock                    | User unlocked an existing session                                                                     |
| **8**  | Network cleartext         | Network logon where credentials were passed to the authentication package in cleartext form           |
| **9**  | New credentials (`runas`) | Process uses different credentials for outbound network connections                                   |
| **10** | RemoteInteractive (RDP)   | Remote Desktop login — check source IP and account                                                    |
| **11** | Cached interactive        | Interactive login using cached domain credentials, typically when a domain controller isn't available |


---

## Account & Group Changes

| ID       | Meaning                                                 | What I'd check nex                                                             |
| -------- | ------------------------------------------------------- |------------------------------------------------------------------------------- |
| **4688** | Process created (command line only if auditing enabled) | Check process name, parent process, command line, user, host, timestamp        |
| **4697** | Service installed (Security log)                        | Check service name, executable path, account, and who installed it             |
| **7045** | Service installed (System log)                          | Check service name, binary path, start type, and installation time**           |
| **4698** | Scheduled task created                                  | Check task name, creator, command/action, trigger, and execution account       |
| **4699** | Scheduled task deleted                                  | Check who deleted it and what the task was doing before deletion               |
| **4702** | Scheduled task updated                                  | Check what changed, who changed it, and what command/action it now runs        |


---

## Execution & Persistence

| ID       | Meaning                                                 | What I'd check next                                                   |
| -------- | ------------------------------------------------------- | --------------------------------------------------------------------- |
| **4688** | Process created (command line only if auditing enabled) | Check **process, parent process, command line, user, host, and time** |
| **4697** | Service installed (Security log)                        | Check **service name, executable path, account, and installer**       |
| **7045** | Service installed (System log)                          | Check **service name, binary path, account, and installation time**   |
| **4698** | Scheduled task created                                  | Check **task name, creator, action/command, trigger, and account**    |
| **4699** | Scheduled task deleted                                  | Check **who deleted it and what the task executed**                   |
| **4702** | Scheduled task updated                                  | Check **what changed, who changed it, and the new action/command**    |


---

## Log Tampering & Policy

| ID       | Meaning                            | What I'd check next                                                                     |
| -------- | ---------------------------------- | --------------------------------------------------------------------------------------- |
| **1102** | Security audit log was cleared     | Check **who cleared it, host, time, and activity immediately before/after**             |
| **4719** | System audit policy was changed    | Check **what policy changed, who changed it, and whether logging was reduced/disabled** |
| **104**  | Event log was cleared (System log) | Check **who cleared it, which host, time, and related activity**                        |

---

## Sysmon — what Security logs don't give you

| ID           | Meaning                                           | What I'd check next                                                                   |
| ------------ | ------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **1**        | Process creation (reliable command line + hashes) | Check process, parent process, command line, user, hash, and time**
| **3**        | Network connection                                | Check source/destination IP, port, process, user, and time
| **7**        | Image / DLL loaded                                | Check which process loaded it, DLL path, and whether it's unusual
| **8**        | CreateRemoteThread                                | Check source/target process, process paths, user, and possible injection activity 
| **11**       | File created                                      | Check file path, creating process, user, and timestamp
| **12/13/14** | Registry create / set / rename                    | Check registry key, value, process, user, and what changed
| **22**       | DNS query                                         | Check **queried domain, process, user, host, and timing


**Sysmon vs Security log — one line:**
Security logs tell you about authentication and security events; Sysmon gives detailed endpoint activity such as processes, files, registry changes, network connections, and DNS queries.

---

## Locations & Tooling

- Logs on disk: `C:\Windows\System32\winevt\Logs\*.evtx`
- Channels: **Security** (auth, account, privilege) · **System** (drivers, services) · **Application** (per-app events)
- `Get-WinEvent -Path <file.evtx>` — reads exported logs from a machine you're not on
- `Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624; StartTime=...}`
- `wevtutil qe Security /c:10 /f:text`

---

## Patterns worth recognising
Many 4625 → one 4624 from the same source
→ Possible password guessing followed by successful login.
4624 Type 10 from an unexpected IP
→ Investigate unusual RDP access.
4720 → 4728/4732
→ New account created and added to a security group.
4720 → 4624
→ Newly created account was subsequently used.
4624 → 4672
→ Successful login followed by special privileges.
4688 → suspicious PowerShell
→ Investigate process execution, parent process, and command line.
4688 → 4698
→ Process activity followed by scheduled-task creation.
4688 → 7045
→ Process activity followed by service installation.
4719 → 1102
→ Audit policy change followed by Security log clearing.
Sysmon 1 → 3 → 22
→ Process creation followed by network connection and DNS activity.
Sysmon 1 → 11
→ Process created a file; investigate the file, hash, and creating process.