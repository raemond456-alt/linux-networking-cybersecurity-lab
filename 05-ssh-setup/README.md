\# SSH Setup



\## Overview



This section documents the setup and testing of Secure Shell (SSH) in my Linux cybersecurity lab.



SSH is a network protocol that allows a user to securely access and manage a remote computer through a command-line interface.



In this lab, I configured Ubuntu to accept SSH connections and used Kali Linux as the client to connect to the Ubuntu machine.



\---



\## Lab Environment



The lab consisted of two virtual machines:



\* \*\*Kali Linux\*\* — SSH client

\* \*\*Ubuntu Linux\*\* — SSH server

\* \*\*Virtualization:\*\* VMware

\* \*\*Network:\*\* Virtual network connecting both machines



The goal was to establish a working SSH connection between Kali Linux and Ubuntu.



\---



\## Objective



The objectives of this practical were to:



1\. Understand how SSH works.

2\. Set up an SSH server on Ubuntu.

3\. Identify the Ubuntu machine's IP address.

4\. Verify that the SSH service is running.

5\. Connect to Ubuntu remotely from Kali Linux.

6\. Understand the initial SSH host verification process.

7\. Prepare the environment for SSH key-based authentication.



\---



\## 1. Setting Up the SSH Server



Ubuntu was configured as the SSH server.



The OpenSSH server provides the service that allows other computers to connect to Ubuntu remotely.



After installing and configuring the SSH service, I verified that the service was running and ready to accept connections.



\---



\## 2. Finding the Ubuntu IP Address



To connect to Ubuntu from Kali Linux, I first needed to identify the IP address assigned to the Ubuntu virtual machine.



The following command can be used:



```bash

ip addr

```



This displays the network interfaces and IP addresses configured on the Ubuntu machine.



The IP address of the Ubuntu machine was then used as the destination when establishing the SSH connection from Kali Linux.



\---



\## 3. Verifying the SSH Service



The SSH service needs to be running before a remote connection can be established.



The listening network sockets can be checked with:



```bash

ss -tulpn

```



This can be used to verify whether the SSH service is listening for incoming connections.



SSH normally uses \*\*TCP port 22\*\*.



\---



\## 4. Connecting From Kali Linux



After confirming the Ubuntu IP address and SSH service, I attempted to connect from Kali Linux.



The general SSH connection format is:



```bash

ssh <username>@<ubuntu-ip-address>

```



For example:



```bash

ssh raymond-favour@<ubuntu-ip-address>

```



The username and IP address should be replaced with the appropriate Ubuntu account and current lab IP address.



\---



\## 5. SSH Host Verification



During the first connection to a new SSH server, SSH may display a message asking whether the server's authenticity should be trusted.



The message includes an ED25519 host key fingerprint.



This is an important security feature because it allows the client to verify the identity of the server before continuing.



After confirming that the server was the intended Ubuntu machine, the host was accepted.



The server's identity is then stored by the SSH client so that future connections can be checked against the known host information.



\---



\## 6. Successful SSH Connection



The Kali Linux machine was successfully able to connect to the Ubuntu machine using SSH.



This confirmed that:



\* The two virtual machines could communicate over the network.

\* Ubuntu was reachable from Kali.

\* The SSH service was running.

\* The SSH server was accepting remote connections.

\* The correct Ubuntu user account could be accessed remotely.



This established the basic SSH communication required for the next stage of the lab.



\---



\## 7. Troubleshooting



During the broader SSH lab, I also practiced troubleshooting common connection problems.



Useful commands included:



```bash

ip addr

```



for checking IP addresses and network interfaces.



```bash

ss -tulpn

```



for checking listening network services.



```bash

journalctl

```



for examining system and service logs.



These tools helped with identifying whether a problem was related to networking, the SSH service, or authentication.



\---



\## What I Learned



Through this practical, I learned:



\* What SSH is and why it is used.

\* The difference between an SSH client and SSH server.

\* How Linux machines can be remotely accessed using SSH.

\* How IP addresses are used to establish remote connections.

\* That SSH normally communicates through TCP port 22.

\* How first-time SSH host verification works.

\* How to perform basic SSH troubleshooting.

\* How SSH provides the foundation for secure remote administration.



\---



\## Next Step



After establishing basic SSH connectivity, the next stage of the lab focused on \*\*SSH key-based authentication\*\*.



Key-based authentication allows a client to authenticate to an SSH server using a cryptographic key pair instead of relying solely on a password.



This was later tested successfully between Kali Linux and Ubuntu.



