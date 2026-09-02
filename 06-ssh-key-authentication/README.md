\# SSH Key-Based Authentication



\## Overview



This section documents the configuration and testing of SSH key-based authentication between my Kali Linux and Ubuntu virtual machines.



SSH key-based authentication allows a user to authenticate to a remote SSH server using a cryptographic key pair instead of relying only on a password.



In this lab, Kali Linux was used as the SSH client and Ubuntu was used as the SSH server.



\---



\## Objective



The objectives of this practical were to:



1\. Understand SSH public-key authentication.

2\. Generate an SSH key pair.

3\. Understand the difference between a private key and a public key.

4\. Transfer the public key to the Ubuntu SSH server.

5\. Configure the `authorized\_keys` file.

6\. Test SSH authentication using the key.

7\. Understand why the private key must remain protected.



\---



\## Lab Environment



| Component    | Role                            |

| ------------ | ------------------------------- |

| Kali Linux   | SSH client                      |

| Ubuntu Linux | SSH server                      |

| VMware       | Virtualization platform         |

| Network      | Virtual network between the VMs |

| Key type     | ED25519                         |



\---



\## 1. Understanding SSH Key Pairs



SSH key authentication uses a pair of mathematically related keys:



```text

SSH Key Pair

│

├── Private Key

│     └── Kept secret on the client

│

└── Public Key

&#x20;     └── Stored on the remote server

```



The two keys work together during authentication.



\### Private Key



The private key is the secret part of the key pair.



It remains on the Kali Linux machine and should never be shared or uploaded to GitHub.



Example filename:



```text

id\_ed25519

```



\### Public Key



The public key can be placed on the SSH server.



Example filename:



```text

id\_ed25519.pub

```



The public key is not a secret and is used by the server to verify that the connecting client possesses the corresponding private key.



\---



\## 2. Generating the SSH Key Pair



An ED25519 SSH key pair was generated on Kali Linux.



The general command for generating an ED25519 key is:



```bash

ssh-keygen -t ed25519

```



This creates two important files:



```text

\~/.ssh/id\_ed25519

\~/.ssh/id\_ed25519.pub

```



The first file is the private key, while the second is the public key.



During key generation, a passphrase can also be configured to provide an additional layer of protection for the private key.



\---



\## 3. Protecting the Private Key



The private key is the most sensitive part of the SSH authentication process.



It should:



\* Remain on the client machine.

\* Never be uploaded to GitHub.

\* Never be sent to another person.

\* Be protected with an appropriate passphrase.

\* Have appropriate file permissions.



Only the public key should be transferred to the SSH server.



\---



\## 4. Copying the Public Key to Ubuntu



The public key was transferred from Kali Linux to the Ubuntu SSH server.



A common way to perform this is:



```bash

ssh-copy-id username@ubuntu-ip

```



The command adds the client's public key to the remote user's SSH authorization file.



The important file on Ubuntu is:



```text

\~/.ssh/authorized\_keys

```



This file contains public keys that are allowed to authenticate as that user.



\---



\## 5. Understanding `authorized\_keys`



The `authorized\_keys` file is located inside the SSH directory of the Ubuntu user.



Conceptually:



```text

Ubuntu

│

└── \~/.ssh/

&#x20;     │

&#x20;     └── authorized\_keys

```



When an SSH client attempts key-based authentication, the SSH server checks whether the client can prove possession of a private key corresponding to one of the public keys stored in `authorized\_keys`.



\---



\## 6. Testing Key-Based Authentication



After transferring the public key to Ubuntu, an SSH connection was initiated from Kali Linux.



The general connection format is:



```bash

ssh username@ubuntu-ip

```



The connection was successfully established from Kali Linux to Ubuntu using the SSH key.



This demonstrated that the public/private key pair was correctly configured.



\---



\## 7. How the Authentication Works



The authentication process can be simplified as follows:



```text

KALI LINUX                         UBUNTU

SSH CLIENT                         SSH SERVER

&#x20;   │                                  │

&#x20;   │  Connection request              │

&#x20;   ├─────────────────────────────────>│

&#x20;   │                                  │

&#x20;   │  Authentication challenge        │

&#x20;   │<─────────────────────────────────┤

&#x20;   │                                  │

&#x20;   │  Proof using private key         │

&#x20;   ├─────────────────────────────────>│

&#x20;   │                                  │

&#x20;   │  Public key checked against      │

&#x20;   │  authorized\_keys                 │

&#x20;   │                                  │

&#x20;   │  Authentication accepted         │

&#x20;   │<─────────────────────────────────┤

&#x20;   │                                  │

&#x20;   │       SSH SESSION ESTABLISHED    │

```



The private key itself is not sent to the server.



Instead, SSH uses cryptographic authentication to prove that the client possesses the corresponding private key.



\---



\## 8. Password Authentication vs Key Authentication



| Password Authentication                        | Key-Based Authentication                     |

| ---------------------------------------------- | -------------------------------------------- |

| Uses a password                                | Uses a cryptographic key pair                |

| Password must be entered during authentication | Private key is used for authentication       |

| Can be vulnerable to password attacks          | Generally stronger when properly configured  |

| Easier to set up                               | Requires initial key configuration           |

| Common for basic SSH access                    | Common for servers and secure administration |



Key-based authentication is particularly useful for managing Linux servers and automating secure connections.



\---



\## 9. Security Considerations



SSH keys provide strong authentication, but they still need to be protected properly.



Important security practices include:



\* Never share the private key.

\* Never commit private keys to Git repositories.

\* Use a passphrase to protect sensitive private keys.

\* Use appropriate permissions on SSH files.

\* Remove unauthorized public keys from `authorized\_keys`.

\* Disable unnecessary authentication methods when appropriate.

\* Keep the SSH server updated.



\---



\## 10. Practical Result



The SSH key-authentication setup was successfully tested.



The final result was:



```text

Kali Linux

&#x20;    │

&#x20;    │ SSH using key authentication

&#x20;    ▼

Ubuntu Linux

&#x20;    │

&#x20;    └── Authentication successful

```



This confirmed that Kali Linux could remotely access the Ubuntu machine using SSH key-based authentication.



\---



\## What I Learned



Through this practical, I learned:



\* How SSH public-key authentication works.

\* The difference between public and private keys.

\* How to generate an ED25519 SSH key pair.

\* Why the private key must remain secret.

\* How `ssh-copy-id` is used to transfer a public key.

\* The purpose of the `authorized\_keys` file.

\* How SSH proves possession of a private key without sending the private key to the server.

\* Why key-based authentication is useful for secure Linux administration.



\---



\## Next Step



After successfully configuring SSH key-based authentication, the next part of the lab focused on \*\*SSH troubleshooting\*\*.



The troubleshooting section documents how I investigated SSH connection failures, network problems, service status, authentication issues, and SSH logs.



