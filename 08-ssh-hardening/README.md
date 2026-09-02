\# 08 - SSH Hardening



\## Objective



The objective of this exercise was to improve the security of the Ubuntu SSH server by reducing unnecessary access to privileged accounts while making sure legitimate SSH access remained functional.



The first hardening change was to disable direct SSH login for the root account.



\---



\## Lab Environment



\* Host OS: Windows

\* Virtualization: VMware

\* Attacker/Client VM: Kali Linux

\* SSH Server VM: Ubuntu 26.04 LTS

\* Ubuntu username: `raymond-favour`

\* Ubuntu IP used during testing: `192.168.223.131`

\* SSH port: `22`



\---



\## 1. Establishing a Baseline



Before making any changes, I checked the current user and SSH service.



```bash

whoami

```



Result:



```text

raymond-favour

```



I then checked the SSH service:



```bash

sudo systemctl status ssh

```



The service was active and running.



The SSH server was listening on port 22 for both IPv4 and IPv6.



The SSH logs also showed that both password authentication and public-key authentication had previously been successful.



\---



\## 2. Inspecting the SSH Configuration



The main SSH server configuration file is:



```text

/etc/ssh/sshd\_config

```



I first displayed the active-looking configuration lines while hiding comments and blank lines:



```bash

sudo grep -vE '^\\s\*#|^\\s\*$' /etc/ssh/sshd\_config

```



The configuration included:



```text

Include /etc/ssh/sshd\_config.d/\*.conf

KbdInteractiveAuthentication no

UsePAM yes

X11Forwarding yes

PrintMotd no

AcceptEnv LANG LC\_\* COLORTERM NO\_COLOR

Subsystem sftp /usr/lib/openssh/sftp-server

```



I also checked the additional configuration directory:



```bash

ls -la /etc/ssh/sshd\_config.d/

```



The directory contained no additional `.conf` files.



\---



\## 3. Checking the Effective SSH Configuration



Instead of relying only on the configuration file, I used:



```bash

sudo sshd -T

```



to inspect the effective SSH configuration.



The relevant settings before hardening were:



```text

port 22

usepam yes

permitrootlogin prohibit-password

pubkeyauthentication yes

passwordauthentication yes

kbdinteractiveauthentication no

x11forwarding yes

```



This showed that:



\* SSH was using port 22.

\* Public-key authentication was enabled.

\* Password authentication was enabled.

\* Root password authentication was already restricted.

\* Keyboard-interactive authentication was disabled.

\* X11 forwarding was enabled.



\---



\## 4. Creating a Configuration Backup



Before modifying the SSH configuration, I created a backup:



```bash

sudo cp /etc/ssh/sshd\_config /etc/ssh/sshd\_config.backup

```



I verified the backup with:



```bash

ls -l /etc/ssh/sshd\_config\*

```



The backup was created successfully.



This provides a recovery copy of the original configuration if a configuration change causes a problem.



\---



\## 5. Disabling Root SSH Login



The configuration contained the commented setting:



```text

\#PermitRootLogin prohibit-password

```



I changed it to:



```text

PermitRootLogin no

```



This explicitly disables SSH login for the root account.



The purpose of this change is to prevent direct remote SSH access to the highly privileged root account.



Instead, users should connect using their normal account and use `sudo` when administrative privileges are required.



\---



\## 6. Validating the Configuration



After making the change, I tested the SSH configuration before reloading the service:



```bash

sudo sshd -t

```



The command returned no output, indicating that there were no SSH configuration syntax errors.



I then checked the effective setting:



```bash

sudo sshd -T | grep '^permitrootlogin'

```



The result was:



```text

permitrootlogin no

```



This confirmed that the intended hardening setting was being applied by the SSH server.



\---



\## 7. Reloading SSH



After validating the configuration, I reloaded the SSH service:



```bash

sudo systemctl reload ssh

```



The command completed without producing an error.



Reloading allows SSH to reread its configuration without unnecessarily stopping the service.



\---



\## 8. Testing SSH After Hardening



I kept the existing Ubuntu session available as a safety measure and created a new SSH connection from Kali Linux.



From Kali:



```bash

ssh raymond-favour@192.168.223.131

```



The SSH client requested the passphrase for the private key:



```text

Enter passphrase for key '/home/kali/.ssh/id\_ed25519':

```



After entering the key passphrase, the connection succeeded and Ubuntu displayed:



```text

Welcome to Ubuntu 26.04 LTS

```



The session opened successfully as:



```text

raymond-favour@raymond-favour-VMware-Virtual-Platform:\~$

```



This confirmed that the hardening change did not break normal key-based SSH access.



\---



\## 9. Security Result



Before the change:



```text

Root SSH login

&#x20;     ↓

prohibit-password

```



After the change:



```text

Root SSH login

&#x20;     ↓

disabled

```



Normal key-based access remained available:



```text

Kali Linux

&#x20;   ↓

SSH

&#x20;   ↓

Ubuntu

&#x20;   ↓

raymond-favour

&#x20;   ↓

SSH key authentication

&#x20;   ↓

Successful login

```



\---



\## 10. Troubleshooting During Configuration



While opening the SSH configuration with Nano, the editor reported that the file was already being edited.



The reported process ID was checked with:



```bash

ps -p 5710 -f

```



No running process was found.



A stale Nano swap file was then identified:



```text

/etc/ssh/.sshd\_config.swp

```



After confirming that the Nano process was no longer running, the stale swap file was removed:



```bash

sudo rm /etc/ssh/.sshd\_config.swp

```



The configuration file could then be opened normally.



This was a useful example of troubleshooting an editor lock rather than immediately forcing the file open.



\---



\## 11. Lessons Learned



\* SSH hardening should be performed carefully because incorrect configuration can cause loss of remote access.

\* A configuration backup should be created before making changes.

\* `sshd -t` can be used to validate SSH configuration syntax before applying changes.

\* `sshd -T` can be used to inspect the effective SSH configuration.

\* Configuration changes should be tested from a new SSH session.

\* Keeping an existing administrative session open provides a recovery path while testing SSH changes.

\* Disabling direct root SSH login reduces the remote attack surface.

\* Key-based authentication provides a way to maintain normal SSH access while strengthening authentication controls.



\---



\## Current Status



Completed:



\* \[x] SSH baseline established

\* \[x] SSH configuration inspected

\* \[x] Additional SSH configuration directory checked

\* \[x] SSH configuration backed up

\* \[x] Direct root SSH login disabled

\* \[x] SSH configuration syntax validated

\* \[x] Effective configuration verified

\* \[x] SSH service reloaded

\* \[x] New Kali → Ubuntu SSH key login tested successfully



Next planned step:



\* Review and safely harden password-based SSH authentication.

\* Validate the configuration before applying further changes.

\* Test key-based authentication again before disabling password authentication.

\## Password Authentication Hardening



After disabling root SSH login, password-based SSH authentication was reviewed and hardened.



\### Initial Configuration



The effective SSH configuration initially showed:



```text

passwordauthentication yes

```



This meant users could still authenticate to SSH using passwords.



\### Configuration Change



The following setting was changed in `/etc/ssh/sshd\_config`:



```text

PasswordAuthentication no

```



During the first validation attempt, a typo was detected:



```text

unsupported option "n0"

```



The configuration was corrected from `n0` to `no`.



\### Configuration Validation



The SSH configuration was tested before reloading the service:



```bash

sudo sshd -t

```



No output was returned, indicating that the configuration syntax was valid.



The effective configuration was then checked:



```bash

sudo sshd -T | grep '^passwordauthentication'

```



Result:



```text

passwordauthentication no

```



\### Reloading SSH



The SSH service was reloaded safely:



```bash

sudo systemctl reload ssh

```



The effective configuration was checked again and confirmed:



```text

passwordauthentication no

```



\### Final Hardening Verification



The final SSH authentication settings were verified with:



```bash

sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication)'

```



Result:



```text

permitrootlogin no

pubkeyauthentication yes

passwordauthentication no

kbdinteractiveauthentication no

```



This confirms that:



\* Root SSH login is disabled.

\* Password-based SSH authentication is disabled.

\* Public-key authentication remains enabled.

\* Keyboard-interactive authentication is disabled.



\### SSH Service Verification



The SSH listening ports were checked with:



```bash

sudo ss -tulpn | grep ':22'

```



SSH was confirmed to be listening on port 22 for both IPv4 and IPv6.



The service status was also checked:



```bash

sudo systemctl status ssh --no-pager

```



The service was confirmed to be:



```text

Active: active (running)

```



The SSH logs also confirmed successful ED25519 public-key authentication:



```text

Accepted publickey for raymond-favour

```



\### Final Status



SSH hardening was successfully completed and verified.



The server now uses key-based authentication while root login and password-based SSH authentication are disabled.



