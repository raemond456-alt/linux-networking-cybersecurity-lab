# Lab 13 - Network Scanning & Service Enumeration Using Nmap
## Overview
This lab focused on using Nmap to perform network scanning and service enumeration against an Ubuntu virtual machine in a controlled VMware laboratory environment.
The objective was to identify remotely accessible TCP ports, determine the services running on those ports, detect service versions, and compare Nmap's remote findings with the services actually listening on the Ubuntu system.
All scanning was performed against virtual machines owned and controlled by me.
## Lab Environment
| Component        | Details                |
| ---------------- | ---------------------- |
| Host OS          | Windows 11             |
| Virtualization   | VMware Workstation     |
| Attacker/Scanner | Kali Linux             |
| Kali IP          | 192.168.223.129        |
| Target           | Ubuntu Linux           |
| Ubuntu IP        | 192.168.223.131        |
| Network          | VMware Virtual Network |
| Scanning Tool    | Nmap 7.99              |
| Target Protocol  | TCP                    |
## Network Topology
```text
Kali Linux
192.168.223.129
      |
      | VMware Virtual Network
      |
      v
Ubuntu Linux
192.168.223.131
      |
      +---- TCP/22 SSH
```
## 1. Verifying Network Connectivity
Before performing the scan, I verified that the Ubuntu target was reachable from Kali.
Command:
```bash
ping -c 4 192.168.223.131
```
The test returned four successful replies with 0% packet loss.
The average round-trip time was approximately 2.398 ms.
This confirmed that the Kali and Ubuntu virtual machines could communicate successfully across the VMware virtual network.
## 2. Performing a Basic Nmap Scan
I performed a basic Nmap scan against the Ubuntu target.
Command:
```bash
nmap 192.168.223.131
```
Result:
```text
PORT   STATE SERVICE
22/tcp open  ssh
```
Nmap reported that the host was up and identified TCP port 22 as open.
The default Nmap scan checks the 1,000 most common TCP ports. In this scan, 999 ports were reported as closed and TCP port 22 was open.
### Interpretation
TCP port 22 was remotely accessible and identified as SSH.
This provided the first indication that the Ubuntu system was exposing an SSH service to the Kali machine.
## 3. Service and Version Detection
To obtain more information about the service running on the open port, I used Nmap service and version detection.
Command:
```bash
nmap -sV 192.168.223.131
```
Result:
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.6 (Ubuntu Linux; protocol 2.0)
```
Nmap also identified the operating system family as Linux.
### Interpretation
The scan identified:
\* Service: SSH
\* Software: OpenSSH
\* Version: 10.2p1 Ubuntu 2ubuntu3.6
\* Protocol: SSH version 2.0
\* Operating system family: Linux
Service and version detection is useful because knowing the software and version can help security analysts determine what technologies are exposed and what further assessment may be appropriate.
The version information alone does not prove that the service is vulnerable.
## 4. Validating Open Ports on Ubuntu
After identifying TCP port 22 remotely, I checked the Ubuntu machine directly to see which TCP services were listening.
Command:
```bash
sudo ss -tlnp
```
Relevant output included:
```text
LISTEN  0  4096  0.0.0.0:22    0.0.0.0:\*    users:(("sshd",pid=6742,fd=3),("systemd",pid=1,fd=480))
LISTEN  0  4096  \[::]:22       \[::]:\*       users:(("sshd",pid=6742,fd=4),("systemd",pid=1,fd=481))
```
This confirmed that the SSH service was listening on port 22.
The service was bound to:
```text
0.0.0.0:22
```
and:
```text
\[::]:22
```
This means SSH was listening on the system's network interfaces for IPv4 and IPv6 connections.
The local listening socket matched the Nmap finding that TCP port 22 was remotely accessible.
## 5. Comparing Local and Remote Service Exposure
The local socket information also showed other services listening on Ubuntu.
For example, CUPS was listening on:
```text
127.0.0.1:631
```
and:
```text
\[::1]:631
```
These addresses represent the local loopback interfaces.
To determine whether port 631 was remotely accessible from Kali, I performed a targeted Nmap scan.
Command:
```bash
nmap -p 631 192.168.223.131
```
Result:
```text
PORT    STATE  SERVICE
631/tcp closed ipp
```
### Interpretation
This result demonstrated an important networking concept:
A service can be running on a machine without being remotely accessible.
CUPS was listening locally on the Ubuntu system, but it was bound to the loopback interface rather than the network interface used by Kali.
Therefore, Nmap reported TCP port 631 as closed from the remote Kali machine.
This comparison helped demonstrate the difference between:
\* A service listening locally
\* A service exposed to the network
## 6. Scanning a Specific Port
I performed a targeted scan of TCP port 22.
Command:
```bash
nmap -p 22 192.168.223.131
```
Result:
```text
PORT   STATE SERVICE
22/tcp open  ssh
```
This confirmed that TCP port 22 was accessible from Kali.
Targeted port scans are useful when an analyst wants to quickly verify the state of a known service without scanning the entire default port range.
## 7. Default NSE Scripts and Service Enumeration
I combined default Nmap scripts with service and version detection.
Command:
```bash
nmap -sC -sV 192.168.223.131
```
Result:
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.6 (Ubuntu Linux; protocol 2.0)
```
### Command breakdown
`-sC`
Runs Nmap's default NSE scripts.
`-sV`
Attempts to identify the service and software version running on discovered ports.
The scan did not produce a large amount of additional script output because the target exposed only the SSH service within the scanned default port range.
## 8. Scanning All TCP Ports
The default Nmap scan checks the 1,000 most common TCP ports.
To perform a broader assessment, I scanned all TCP ports from 1 through 65535.
Command:
```bash
nmap -p- 192.168.223.131
```
Result:
```text
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
```
### Interpretation
The full TCP scan found:
\* 65,534 closed TCP ports
\* 1 open TCP port
\* TCP port 22 running SSH
This provided stronger evidence that SSH was the only remotely accessible TCP service on the target across the full TCP port range.
## 9. Targeted SSH Service Enumeration
After identifying SSH as the only open TCP service, I performed a targeted service/version scan against port 22.
Command:
```bash
nmap -p 22 -sV 192.168.223.131
```
Result:
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.6 (Ubuntu Linux; protocol 2.0)
```
This confirmed the SSH service and its detected version.
Targeting a known port can make enumeration more focused and easier to interpret.
## 10. Key Findings
The scans produced the following findings:
| Test                       | Result                           |
| -------------------------- | -------------------------------- |
| Network connectivity       | Successful                       |
| Default TCP scan           | TCP/22 open                      |
| Service detection          | OpenSSH 10.2p1 Ubuntu 2ubuntu3.6 |
| Port 22 targeted scan      | Open                             |
| Port 631 remote scan       | Closed                           |
| Default NSE + version scan | SSH identified                   |
| Full TCP scan              | Only TCP/22 open                 |
The most significant finding was that TCP port 22 was the only remotely accessible TCP port identified during the full TCP scan.
## 11. Security Observations
### 1. Open ports increase the attack surface
A remotely accessible service provides a potential entry point into a system.
This does not automatically mean the service is vulnerable, but exposed services should be identified and assessed.
### 2. Service exposure depends on network binding
The CUPS service demonstrated that a locally running service does not necessarily have to be remotely accessible.
Binding a service to a loopback address can prevent remote systems from connecting to it.
### 3. Version detection provides useful assessment information
Nmap identified the OpenSSH version running on the Ubuntu target.
Knowing the service and version can help an analyst determine what security checks should be performed next.
However, version detection by itself does not establish that a vulnerability exists.
### 4. Full port scans provide broader visibility
A default Nmap scan checks common ports, while a full TCP scan checks ports 1 through 65535.
The full scan provided stronger evidence that no additional TCP services were exposed on the target.
### 5. Scan results should be validated
The Nmap results were compared with Ubuntu's local listening sockets using `ss`.
This helped confirm why SSH was remotely accessible while CUPS was not.
## 12. Lessons Learned
Through this lab, I learned how to:
\* Verify connectivity between virtual machines before scanning.
\* Perform a basic Nmap TCP scan.
\* Identify open and closed ports.
\* Use `-sV` for service and version detection.
\* Use `-sC` to run Nmap's default NSE scripts.
\* Scan a specific TCP port.
\* Scan the complete TCP port range with `-p-`.
\* Validate network scan results using `ss`.
\* Understand the difference between a locally listening service and a remotely exposed service.
\* Interpret Nmap results from a defensive security perspective.
\* Preserve scan output as evidence for documentation.
## 13. Nmap Evidence Files
The Nmap results were saved on the Kali Linux VM in:
```text
/home/kali/nmap-lab/
```
The following evidence files were created:
```text
basic-scan.txt
full-tcp-scan.txt
port-631-test.txt
service-enumeration.txt
```
These files contain the command output used to support the observations documented in this lab.
The evidence was retained in the Kali laboratory environment rather than added directly to the GitHub repository.
## 14. Ethical and Safety Considerations
All scanning in this lab was performed against virtual machines that I controlled inside a private VMware laboratory network.
Nmap should only be used against systems where the tester has permission to perform security testing.
This lab did not involve scanning public or third-party systems.
## Conclusion
This lab demonstrated a basic network reconnaissance and service enumeration workflow using Nmap.
The Ubuntu target was reachable from Kali, and Nmap identified TCP port 22 as the only remotely accessible TCP port during the full TCP scan.
Service detection identified OpenSSH 10.2p1 Ubuntu 2ubuntu3.6 running on the target.
The comparison with `ss` also demonstrated that services can be running locally without being exposed to remote systems.
The exercise provided practical experience with identifying network exposure, enumerating services, validating scan results, and documenting findings in a controlled cybersecurity laboratory.

