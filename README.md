\# Linux Networking \& Cybersecurity Lab



A hands-on cybersecurity lab documenting my practical learning and experimentation with Linux, networking, SSH, system administration, and basic security concepts.



The purpose of this project is to build practical cybersecurity skills through a controlled virtual lab environment and document the process, commands, troubleshooting steps, and lessons learned.



\---



\## 🎯 Project Goals



\* Build confidence working with Linux systems

\* Understand basic networking concepts

\* Learn how to identify and troubleshoot network connectivity issues

\* Practice network reconnaissance using Nmap

\* Understand how SSH works

\* Configure and secure SSH access

\* Practice Linux permissions and user management

\* Learn how to troubleshoot services and authentication

\* Develop good cybersecurity documentation habits

\* Build a public portfolio of hands-on cybersecurity projects



\---



\## 🖥️ Lab Environment



| Component                  | Details                                     |

| -------------------------- | ------------------------------------------- |

| Host Operating System      | Windows                                     |

| Virtualization             | VMware                                      |

| Attacker / Testing Machine | Kali Linux                                  |

| Target Machine             | Ubuntu Linux                                |

| Network                    | Isolated virtual network                    |

| Remote Access              | SSH                                         |

| Key Authentication         | ED25519                                     |

| Main Tools                 | Nmap, SSH, `ip`, `ping`, `ss`, `journalctl` |



\---



\## 📚 Topics Covered



\### Linux Fundamentals



\* Navigating the Linux terminal

\* Working with users

\* Understanding `whoami`

\* Using `sudo`

\* Understanding root privileges

\* File and directory permissions

\* Basic Linux system administration



\### Networking



\* Understanding IP addresses

\* Using `ip addr`

\* Testing connectivity with `ping`

\* Understanding virtual machine networking

\* Identifying hosts on a network

\* Understanding ports and services



\### Network Reconnaissance



\* Introduction to Nmap

\* Host discovery

\* Port scanning

\* Identifying open ports

\* Service and version detection

\* Understanding the information obtained from network scans



\### SSH



\* Installing and configuring an SSH server

\* Starting and checking SSH services

\* Connecting between Kali Linux and Ubuntu

\* Understanding SSH authentication

\* Generating ED25519 SSH keys

\* Public-key authentication

\* `authorized\_keys`

\* `ssh-copy-id`

\* Troubleshooting SSH connections

\* Checking SSH connection information

\* Reviewing SSH logs with `journalctl`

\* Backing up SSH configuration

\* SSH security hardening



\---



\## 🔐 SSH Authentication



One of the practical exercises in this lab involved configuring SSH key-based authentication between Kali Linux and Ubuntu.



The authentication process uses an ED25519 key pair:



```text

Private Key

&#x20;   ↓

Stored securely on the client



Public Key

&#x20;   ↓

Stored on the Ubuntu SSH server



&#x20;       ↓



Kali Linux ─────── SSH ───────> Ubuntu

&#x20;                Authentication

```



After configuring the public key on the Ubuntu system, I successfully established SSH access from Kali Linux using the key.



\*\*Security note:\*\* Private SSH keys and passwords are never included in this repository.



\---



\## 🧪 Practical Progress



| #  | Practical                        | Status         |

| -- | -------------------------------- | -------------- |

| 01 | VMware and Virtual Machine Setup | ✅ Completed    |

| 02 | Linux Fundamentals               | ✅ Completed    |

| 03 | Basic Networking                 | ✅ Completed    |

| 04 | Nmap Network Scanning            | ✅ Completed    |

| 05 | SSH Setup and Configuration      | ✅ Completed    |

| 06 | SSH Key-Based Authentication     | ✅ Completed    |

| 07 | SSH Troubleshooting              | ✅ Completed    |

| 08 | SSH Security Hardening           | 🔄 In Progress |



\---



\## 🛠️ Skills Practiced



\* Linux command line

\* Linux system administration

\* Networking fundamentals

\* TCP/IP concepts

\* IP addressing

\* Port and service identification

\* Network reconnaissance

\* Nmap

\* SSH

\* Public-key authentication

\* Linux permissions

\* Service management

\* Log analysis

\* Troubleshooting

\* Basic security hardening

\* Git and GitHub documentation



\---



\## 📂 Documentation Structure



Detailed practical documentation will be organized into separate directories:



```text

linux-networking-cybersecurity-lab/

│

├── README.md

│

├── 01-vmware-setup/

│   └── README.md

│

├── 02-linux-fundamentals/

│   └── README.md

│

├── 03-networking/

│   └── README.md

│

├── 04-nmap-scanning/

│   └── README.md

│

├── 05-ssh-setup/

│   └── README.md

│

├── 06-ssh-key-authentication/

│   └── README.md

│

├── 07-ssh-troubleshooting/

│   └── README.md

│

└── 08-ssh-hardening/

&#x20;   └── README.md

```



Each practical will contain:



\* Objective

\* Lab environment

\* Commands used

\* Expected output

\* Actual results

\* Problems encountered

\* Troubleshooting steps

\* Security considerations

\* Lessons learned



\---



\## 📝 Documentation Method



For each practical, I document:



```text

What I wanted to achieve

&#x20;       ↓

Commands / configuration used

&#x20;       ↓

What happened

&#x20;       ↓

Problems encountered

&#x20;       ↓

How I fixed them

&#x20;       ↓

What I learned

```



This approach helps turn the lab into both a learning record and a cybersecurity portfolio.



\---



\## ⚠️ Security \& Ethical Use



All security testing documented in this repository is performed in my own controlled virtual lab environment.



The techniques and tools demonstrated here should only be used on systems and networks where I have explicit authorization to test.



Sensitive information such as:



\* Private SSH keys

\* Passwords

\* Authentication tokens

\* API keys

\* Personal credentials

\* Other secrets



will not be committed to this repository.



\---



\## 🚀 Current Focus



The current stage of the lab focuses on \*\*SSH security hardening\*\*.



The goal is to understand how an SSH service can be configured more securely while maintaining legitimate administrative access.



Future work will expand into additional Linux administration, networking, reconnaissance, and cybersecurity exercises.



\---



\## 📈 Project Status



\*\*Status:\*\* Active



This repository will be continuously updated as I complete additional cybersecurity practicals and develop my skills.



