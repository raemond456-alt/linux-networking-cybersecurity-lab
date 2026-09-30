\# Lab 12 — Linux Log Analysis



\## Overview



This lab focused on analyzing Linux authentication and system logs to understand how SSH activity is recorded.



The investigation was performed in a controlled VMware lab using:



\- Kali Linux — `192.168.223.129`

\- Ubuntu Linux — `192.168.223.131`

\- SSH — TCP port `22`

\- Authentication log — `/var/log/auth.log`

\- System journal — `journalctl`



The main objectives were to:



\- Understand the purpose of Linux authentication logs.

\- Identify successful SSH authentication events.

\- Investigate failed SSH authentication attempts.

\- Identify source IP addresses and source ports.

\- Use `grep` and `awk` to filter and extract useful log information.

\- Correlate SSH events using process IDs (PIDs).

\- Use `journalctl` to investigate SSH service events.

\- Verify the SSH service status after the investigation.



All testing was performed between my own Kali and Ubuntu virtual machines.

## Lab Environment



| Component | Details |

|---|---|

| Host OS | Windows 11 |

| Hypervisor | VMware Workstation |

| Client | Kali Linux |

| Client IP | `192.168.223.129` |

| Server | Ubuntu Linux |

| Server IP | `192.168.223.131` |

| Protocol | SSH |

| SSH Port | `22` |

| Authentication | ED25519 public key |

| Primary Log | `/var/log/auth.log` |

| System Journal | `journalctl` |



\### Network Layout



```text

Kali Linux                              Ubuntu Linux

192.168.223.129                         192.168.223.131

&#x20;      |                                        |

&#x20;      |------------ SSH / TCP 22 ------------>|

&#x20;      |                                        |

&#x20;      |<----------- Authentication ---------->|

&#x20;      |                                        |

&#x20;      VMware Virtual Network```


---



\## 1. Inspecting the Authentication Log



Linux stores many authentication-related events in:



```text

/var/log/auth.log



I first checked the file's permissions, ownership, size, and modification time.



Command

sudo ls -lh /var/log/auth.log

Output

\-rw-r----- 1 syslog adm 26K Sep 28 13:52 /var/log/auth.log

What the command does

sudo — runs the command with elevated privileges.

ls — lists information about a file.

\-l — displays detailed information.

\-h — displays file sizes in a human-readable format.

/var/log/auth.log — the authentication log file being inspected.



The output showed that the file is owned by the syslog user and adm group and had a size of approximately 26 KB at the time of the check.


\---



\## 2. Viewing Recent Authentication Events



After confirming that the authentication log existed, I viewed the most recent entries to get an initial overview of authentication-related activity.



\### Command



```bash

sudo tail -n 20 /var/log/auth.log



\### What the command does



\- `sudo` — runs the command with elevated privileges.

\- `tail` — displays the end of a file.

\- `-n 20` — tells `tail` to display the last 20 lines.

\- `/var/log/auth.log` — the authentication log being examined.

\### Observations 



The recent entries included several types of authentication-related activity, including:



sudo sessions opened and closed by the user.

Scheduled CRON sessions running as root.

Graphical login/keyring activity from gdm-password.

Commands executed with sudo.



These entries provided an initial view of normal authentication and system activity before focusing specifically on SSH events.


## 3. Filtering SSH Activity



After reviewing the recent authentication events, I filtered the authentication log for entries related to SSH.



\### Command



```bash

sudo grep -i ssh /var/log/auth.log

### What the command does



\- `sudo` — runs the command with elevated privileges.

\- `grep` — searches text for a specified pattern.

\- `-i` — makes the search case-insensitive.

\- `ssh` — the text pattern being searched for.

\- `/var/log/auth.log` — the log file being searched.



The command returned several SSH-related events, including the SSH server starting, successful public-key authentication, SSH sessions being opened and closed, and SSH service activity.



These results provided a more focused view of SSH activity within the authentication log.



\## 4. Identifying Successful SSH Authentication



To identify successful SSH logins, I searched the authentication log specifically for Accepted publickey events.



\### Command



```bash

sudo grep "Accepted publickey" /var/log/auth.log



\### What the command does



\- `sudo` — runs the command with elevated privileges.

\- `grep` — searches the log for matching text.

\- `"Accepted publickey"` — searches specifically for successful public-key authentication.

\- `/var/log/auth.log` — the authentication log being examined.



The search returned successful SSH authentication events for the `raymond-favour` account.



The entries showed that the connections originated from the Kali Linux VM at `192.168.223.129` and used ED25519 public-key authentication.



Three relevant successful SSH authentication events were observed:



| Date | User | Source IP | Source Port |

|---|---|---|---|

| 2026-09-23 15:44 | `raymond-favour` | `192.168.223.129` | `45350` |

| 2026-09-23 15:57 | `raymond-favour` | `192.168.223.129` | `54238` |

| 2026-09-25 18:59 | `raymond-favour` | `192.168.223.129` | `55658` |



These entries confirmed that the Ubuntu SSH server successfully authenticated the `raymond-favour` account using a public key from the Kali VM.



\## 5. Extracting Useful Fields with `awk`



The SSH log entries contained several fields of information. To make the output easier to read, I used `awk` to extract the timestamp, username, and source IP address.



\### Command



```bash

sudo grep "Accepted publickey" /var/log/auth.log | awk '{print $1, $7, $9}'



\### What the command does



The command uses a pipe (`|`) to send the output of `grep` directly to `awk`.



\- `grep "Accepted publickey"` — finds successful public-key authentication events.

\- `|` — passes the output of the first command to the next command.

\- `awk` — processes the text and allows specific fields to be selected.

\- `$1` — the first field, containing the timestamp.

\- `$7` — the seventh field, containing the username.

\- `$9` — the ninth field, containing the source IP address.

\- `print` — displays the selected fields.



For a typical SSH authentication entry, the relevant fields were:



```text

$1 = timestamp

$7 = username

$9 = source IP

This produced a simpler view of the SSH authentication events by displaying only the information needed for the investigation.



Why this is useful



Log files can contain a large amount of information. Using tools such as grep and awk makes it easier to isolate important fields during log analysis and investigation.



\## 6. Improving the SSH Log Filter



The initial search for Accepted publickey also matched the grep command that was used to perform the search. This happened because the command itself was recorded in the authentication log by sudo.



To reduce unrelated matches, I refined the search pattern to look for the SSH session process followed by the successful authentication message.



Command

sudo grep -E "sshd-session\\\[\[0-9]+\\]: Accepted publickey" /var/log/auth.log

What the command does

grep — searches the log for matching text.

\-E — enables extended regular expressions.

sshd-session — matches the SSH session process name.

\\\[ and \\] — match the square brackets surrounding the process ID.

\[0-9]+ — matches one or more digits representing the process ID.

Accepted publickey — matches successful public-key authentication events.



The refined filter returned the actual SSH authentication events without the unrelated sudo grep command entries.



This demonstrated the importance of creating specific search patterns when analyzing system logs.



\## 7. Correlating SSH Events Using a Process ID



After identifying successful SSH authentication events, I examined the entries associated with a specific sshd-session process.



I selected process ID 5521 from one of the successful SSH authentication events.



Command

sudo grep "sshd-session\\\[5521\\]" /var/log/auth.log

What the command does

sudo — runs the command with elevated privileges.

grep — searches the authentication log.

"sshd-session\\\[5521\\]" — searches for entries generated by the sshd-session process with PID 5521.

/var/log/auth.log — the authentication log being investigated.

Observed events



The search returned a sequence of events associated with the same sshd-session\[5521] process:



Accepted publickey for raymond-favour from 192.168.223.129

Session opened for user raymond-favour

Child terminated by signal 15

Session closed for user raymond-favour

Interpretation



The events showed the following sequence:



Successful public-key authentication

&#x20;           ↓

SSH session opened

&#x20;           ↓

SSH session process received SIGTERM

&#x20;           ↓

SSH session closed



The Accepted publickey entry shows that authentication succeeded.



The session opened entry indicates that the authenticated SSH session was established.



The child terminated by signal 15 entry indicates that a child process received SIGTERM, a termination signal.



Finally, the session closed entry shows that the SSH session was closed.



The process ID provided a useful way to correlate these related log entries chronologically.



Note: The process ID identifies the sshd-session process that generated these entries. It should not be treated as a permanent identifier for the user's SSH session.





\## 8. Investigating a Failed SSH Authentication Attempt



To observe how a rejected SSH authentication attempt was recorded, I performed a controlled test between my own Kali and Ubuntu virtual machines.



The Ubuntu SSH configuration had password authentication disabled and public-key authentication enabled. Therefore, instead of testing a password, I generated a temporary ED25519 key on Kali that was not authorized on the Ubuntu server.



Checking the SSH Authentication Configuration



Before performing the test, I checked the effective SSH configuration:



sudo sshd -T | grep -E '^(passwordauthentication|pubkeyauthentication)'



The output showed:



pubkeyauthentication yes

passwordauthentication no



This confirmed that the SSH server accepted public-key authentication while password authentication was disabled.



Generating a Temporary Test Key



On Kali Linux, I generated a temporary ED25519 key pair:



ssh-keygen -t ed25519 -f /tmp/test\_failed\_ssh -N ""



The key was created specifically for the controlled test and was not added to the authorized keys on the Ubuntu server.



Attempting SSH Authentication



I then attempted to connect to Ubuntu using only the temporary private key:



ssh -o IdentitiesOnly=yes -i /tmp/test\_failed\_ssh raymond-favour@192.168.223.131



The connection was rejected with:



raymond-favour@192.168.223.131: Permission denied (publickey).



This confirmed that the temporary public key was not authorized by the Ubuntu SSH server.



Investigating the Ubuntu Log



I searched /var/log/auth.log for the Kali VM's IP address:



sudo grep '192.168.223.129' /var/log/auth.log | tail -n 20



The relevant entry was:



2026-09-30T14:07:53.386022+01:00 raymond-favour-VMware-Virtual-Platform sshd-session\[29111]: Connection closed by authenticating user raymond-favour 192.168.223.129 port 54974 \[preauth]



The \[preauth] marker indicates that the connection was still in the pre-authentication stage when it was closed.



The entry showed that:



The connection originated from the Kali VM at 192.168.223.129.

The attempted account was raymond-favour.

The source port was 54974.

The connection was closed before successful authentication.

The event occurred during the pre-authentication stage.



Because password authentication was disabled, this failed attempt did not appear as a Failed password event. Instead, the SSH server recorded the connection closing during the authentication process.



Investigation Result



The controlled test demonstrated that an unauthorized public key was rejected by the SSH server and that the event was recorded in the authentication log.



This shows how authentication logs can provide useful evidence when investigating unsuccessful SSH connection attempts.



\## 9. Correlating the Failed Authentication Event



To examine the failed authentication event more precisely, I searched the authentication log using the sshd-session process ID associated with the connection.



Command

sudo grep 'sshd-session\\\[29111\\]' /var/log/auth.log

Output

2026-09-30T14:07:53.386022+01:00 raymond-favour-VMware-Virtual-Platform sshd-session\[29111]: Connection closed by authenticating user raymond-favour 192.168.223.129 port 54974 \[preauth]

Interpretation



The log entry can be broken down as follows:



Field	Meaning

2026-09-30T14:07:53	Date and time of the event

sshd-session\[29111]	SSH session process and PID

raymond-favour	Account being authenticated

192.168.223.129	Source IP address of the Kali VM

54974	Temporary source TCP port

\[preauth]	Connection was closed before successful authentication



The event correlated with the failed SSH attempt from Kali, which returned Permission denied (publickey).



This provided a clear example of how process IDs, timestamps, usernames, and IP addresses can be used together to investigate an authentication event.



\## 10. Investigating SSH Service Events with journalctl



In addition to /var/log/auth.log, I used journalctl to examine events generated by the SSH service around the time of the controlled authentication test.



Command

sudo journalctl -u ssh --since "2026-09-30 14:07:40" --until "2026-09-30 14:08:05"

What the command does

sudo — runs the command with elevated privileges.

journalctl — displays entries from the systemd journal.

\-u ssh — limits the results to the SSH service.

\--since — specifies the beginning of the time range.

\--until — specifies the end of the time range.



This allowed me to examine SSH service activity within a small time window surrounding the authentication attempt.



Relevant Output



The journal showed:



Sep 30 14:07:53 raymond-favour-VMware-Virtual-Platform systemd\[1]: Starting ssh.service - OpenBSD Secure Shell server...

Sep 30 14:07:53 raymond-favour-VMware-Virtual-Platform sshd\[29109]: Server listening on 0.0.0.0 port 22.

Sep 30 14:07:53 raymond-favour-VMware-Virtual-Platform sshd\[29109]: Server listening on :: port 22.

Sep 30 14:07:53 raymond-favour-VMware-Virtual-Platform systemd\[1]: Started ssh.service - OpenBSD Secure Shell server.

Sep 30 14:07:53 raymond-favour-VMware-Virtual-Platform sshd-session\[29111]: Connection closed by authenticating user raymond-favour 192.168.223.129 port 54974 \[preauth]

Observation



The journal showed that the SSH service was starting at approximately the same time as the controlled authentication attempt.



Because the SSH service was being started or restarted at that time, the test was not completely isolated from other SSH service activity. Therefore, the log evidence confirms the failed pre-authentication connection, but it would not be appropriate to assume that every event occurring at that exact second was caused by the test itself.



This is an important lesson in log analysis: events should be interpreted in their surrounding context rather than in isolation.



\## 11. Verifying the SSH Service Status



After completing the log investigation, I checked the current status of the SSH service to confirm that it was running normally.



Command

sudo systemctl status ssh --no-pager

What the command does

sudo — runs the command with elevated privileges.

systemctl — manages and queries systemd services.

status — displays the current status of a service.

ssh — specifies the OpenSSH server service.

\--no-pager — displays the output directly in the terminal instead of opening it in a pager.

Relevant Output



The service reported:



● ssh.service - OpenBSD Secure Shell server

&#x20;    Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)

&#x20;    Active: active (running)

&#x20;    Process: 29106 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)

&#x20;    Main PID: 29109 (sshd)

Interpretation



The output showed that:



Active: active (running) — the SSH service was running normally.

ExecStartPre=/usr/sbin/sshd -t ... status=0/SUCCESS — the SSH configuration passed the configuration test before the service started.

Main PID: 29109 — the main SSH daemon process was running with PID 29109.

TriggeredBy: ssh.socket — the SSH service can be activated through the SSH socket.



The disabled value shown in the Loaded line refers to the service's startup configuration and does not mean that the SSH service was currently stopped. The Active: active (running) status confirmed that SSH was currently operational.



Result



The final status check confirmed that the SSH server remained operational after the log-analysis investigation.


\## 12. Key Findings



The Linux log analysis investigation produced the following findings:



SSH authentication activity was recorded in /var/log/auth.log.

The log contained successful public-key authentication events, session openings, session closures, and SSH service activity.

Successful SSH authentication was identified.

Multiple successful public-key authentication events were found for the raymond-favour account. The connections originated from the Kali VM at 192.168.223.129.

No matching failed password authentication events were found.

The current authentication log did not contain Failed password or Invalid user events for SSH.



A controlled unauthorized-key test was rejected.

A temporary ED25519 key that was not authorized on the Ubuntu server resulted in:



Permission denied (publickey).



The failed authentication attempt was recorded as a pre-authentication connection closure.

The Ubuntu log recorded:



Connection closed by authenticating user raymond-favour 192.168.223.129 port 54974 \[preauth]

SSH events could be correlated using process IDs.

Searching for specific sshd-session PIDs made it possible to follow related authentication and session events.

grep and awk were useful for log analysis.

grep was used to locate relevant events, while awk was used to extract specific fields such as timestamps, usernames, and source IP addresses.

journalctl provided additional SSH service information.

The system journal was used to examine SSH service activity within a specific time window.

The SSH service remained operational after the investigation.

The final systemctl status ssh check showed that the service was active and running.



\## 13. Lessons Learned



This lab provided practical experience with Linux authentication and system log analysis.



The main lessons learned were:



Linux authentication logs can provide useful evidence about login and authentication activity.

SSH authentication events can reveal the username, source IP address, source port, authentication method, and session activity.

grep can be used to quickly filter large log files for specific events.

awk can extract individual fields from structured log entries.

Process IDs can help correlate related events generated by the same SSH process.

A failed public-key authentication attempt may not appear as a Failed password event when password authentication is disabled.

The \[preauth] marker can indicate that an SSH connection was closed before successful authentication.

journalctl provides another source of information for investigating system and service activity.

Log events should be interpreted together with their timestamps and surrounding context rather than examined in isolation.

Controlled testing in a virtual lab provides a safe way to understand how security events appear in real system logs.

Overall Outcome



By completing this lab, I gained practical experience identifying, filtering, extracting, and correlating SSH authentication events on a Linux system.



The investigation also improved my understanding of how authentication logs can be used as evidence during basic security monitoring and incident investigation.









