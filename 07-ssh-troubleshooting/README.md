\# SSH Troubleshooting



\## Overview



This section documents the troubleshooting techniques I used while working with SSH in my Linux cybersecurity lab.



Troubleshooting SSH requires identifying which part of the connection is failing instead of changing settings randomly.



The troubleshooting process used a layered approach, starting with basic network connectivity and progressing toward the SSH service and authentication.



\---



\## Objective



The objectives of this practical were to:



1\. Understand common SSH connection problems.

2\. Check network connectivity between Kali Linux and Ubuntu.

3\. Verify IP addresses and network interfaces.

4\. Check whether SSH is listening on the expected port.

5\. Verify the SSH service.

6\. Investigate SSH configuration and authentication problems.

7\. Use system logs to identify errors.

8\. Develop a structured troubleshooting methodology.



\---



\## Lab Environment



| Component        | Role                            |

| ---------------- | ------------------------------- |

| Kali Linux       | SSH client                      |

| Ubuntu Linux     | SSH server                      |

| VMware           | Virtualization platform         |

| Network          | Virtual network between the VMs |

| SSH              | Remote administration service   |

| Default SSH port | TCP 22                          |



\---



\## Troubleshooting Methodology



Instead of immediately changing configurations, I learned to troubleshoot SSH systematically.



The general process was:



```text

Network

&#x20;  ↓

IP Address

&#x20;  ↓

Connectivity

&#x20;  ↓

Port

&#x20;  ↓

SSH Service

&#x20;  ↓

SSH Configuration

&#x20;  ↓

Authentication

&#x20;  ↓

Logs

```



This approach makes it easier to identify the exact stage where a problem occurs.



\---



\## 1. Check the IP Address



The first step is to confirm that the Ubuntu machine has a valid IP address.



On Ubuntu:



```bash

ip addr

```



This displays the available network interfaces and their assigned addresses.



The correct IP address is required before Kali can establish an SSH connection to Ubuntu.



\### Why this matters



If the Ubuntu machine has the wrong IP address, or if the expected network interface is unavailable, an SSH connection will fail even if the SSH service itself is working correctly.



\---



\## 2. Test Network Connectivity



After identifying the Ubuntu IP address, basic connectivity can be tested from Kali using:



```bash

ping <ubuntu-ip>

```



A successful response indicates that the two machines can communicate at the network level.



If ping fails, the problem may be related to:



\* Incorrect IP addresses

\* Network configuration

\* VMware networking

\* Disabled network interfaces

\* Firewall rules

\* The target machine being offline



\---



\## 3. Check SSH Port Availability



SSH normally listens on:



```text

TCP port 22

```



Nmap can be used from Kali to examine the target machine:



```bash

nmap <ubuntu-ip>

```



Service and version detection can also provide additional information:



```bash

nmap -sV <ubuntu-ip>

```



This helps determine whether the SSH service is exposed and can provide information about the detected service.



\---



\## 4. Check Listening Services on Ubuntu



On Ubuntu, listening network services can be examined using:



```bash

ss -tulpn

```



This helps determine whether a service is listening for incoming network connections.



For SSH, the important thing to look for is a listening socket associated with port 22.



\---



\## 5. Check the SSH Service



If the SSH server is not running, remote connections will fail.



The SSH service status can be checked using the system service-management tools.



For example:



```bash

systemctl status ssh

```



If the service is stopped, it may need to be started before SSH connections can succeed.



The important troubleshooting question is:



> Is the SSH server actually running?



\---



\## 6. Test the SSH Connection



Once networking and the SSH service have been checked, the connection itself can be tested:



```bash

ssh username@ubuntu-ip

```



Possible problems at this stage can include:



\* Incorrect username

\* Incorrect IP address

\* Authentication failure

\* SSH configuration problems

\* Incorrect permissions

\* Missing public key

\* Password authentication problems



\---



\## 7. Investigating SSH Logs



When the cause of an SSH problem is not immediately obvious, system logs can provide useful information.



The `journalctl` command can be used to inspect system and service logs:



```bash

journalctl

```



The logs can help identify events related to:



\* SSH service failures

\* Authentication attempts

\* Permission problems

\* Service startup issues

\* Configuration errors



Logs are particularly useful because they provide information about what the system is actually doing rather than requiring assumptions.



\---



\## 8. Common SSH Problems



\### Problem: Connection Refused



A connection-refused error can indicate that:



\* The SSH service is not running.

\* SSH is not listening on the expected port.

\* A firewall or network rule is blocking the connection.



The first things to check are:



```bash

systemctl status ssh

```



and:



```bash

ss -tulpn

```



\---



\### Problem: Connection Timeout



A timeout can indicate a network or connectivity problem.



Useful checks include:



```bash

ip addr

```



and:



```bash

ping <ubuntu-ip>

```



Nmap can also be used to investigate whether the target is reachable and whether the expected ports are available.



\---



\### Problem: Permission Denied



If the server is reachable but authentication fails, possible causes include:



\* Incorrect username

\* Incorrect password

\* Incorrect SSH key

\* Public key not present in `authorized\_keys`

\* Incorrect permissions on SSH files

\* SSH configuration restrictions



This requires investigating the authentication configuration rather than the network connection.



\---



\### Problem: Host Key Warning



When connecting to a server for the first time, SSH may display a host authenticity warning.



This is normal behavior.



The SSH client is asking the user to verify the server's host key before trusting it.



The host key information is then stored by the SSH client for future verification.



\---



\## 9. A Structured Troubleshooting Checklist



When an SSH connection fails, I can use the following checklist:



```text

\[ ] Is Ubuntu powered on?

&#x20;       ↓

\[ ] Does Ubuntu have the expected IP address?

&#x20;       ↓

\[ ] Can Kali reach the Ubuntu IP?

&#x20;       ↓

\[ ] Is TCP port 22 available?

&#x20;       ↓

\[ ] Is the SSH service running?

&#x20;       ↓

\[ ] Is SSH listening on the expected interface/port?

&#x20;       ↓

\[ ] Is the SSH configuration correct?

&#x20;       ↓

\[ ] Is the username correct?

&#x20;       ↓

\[ ] Is authentication configured correctly?

&#x20;       ↓

\[ ] Do the system logs show an error?

```



This prevents troubleshooting from becoming random trial and error.



\---



\## What I Learned



Through this practical, I learned that SSH troubleshooting should be performed systematically.



I learned how to:



\* Check Linux network interfaces with `ip addr`.

\* Test connectivity using `ping`.

\* Investigate network services using Nmap.

\* Check listening sockets with `ss`.

\* Check the SSH service status.

\* Investigate system logs with `journalctl`.

\* Distinguish between network problems and authentication problems.

\* Use a structured troubleshooting process rather than changing configurations randomly.



\---



\## Key Lesson



One of the most important lessons from this practical was:



> \*\*Troubleshoot from the bottom up.\*\*



If the network connection itself is broken, there is no point troubleshooting SSH authentication yet.



A reliable troubleshooting sequence is:



```text

Network

&#x20;  ↓

Connectivity

&#x20;  ↓

Port

&#x20;  ↓

Service

&#x20;  ↓

Configuration

&#x20;  ↓

Authentication

&#x20;  ↓

Logs

```



This methodology can also be applied to troubleshooting many other network services.



\---



\## Next Step



The next stage of the lab is \*\*SSH Hardening\*\*.



After learning how SSH works, how key-based authentication works, and how to troubleshoot SSH problems, the next objective is to improve the security of the SSH server by reducing unnecessary attack surface and strengthening its configuration.



