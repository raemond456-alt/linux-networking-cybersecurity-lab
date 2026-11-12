# Lab 14: Linux Firewall Configuration and Network Traffic Filtering

## 1. Objective

Configure and enable the Uncomplicated Firewall (UFW) on Ubuntu, apply default firewall policies, allow SSH access, and verify traffic filtering from Kali Linux with Nmap.

## 2. Lab Environment

| Component | Details |
|---|---|
| Host System | Windows 11 |
| Virtualization Platform | VMware Workstation |
| Target Machine | Ubuntu |
| Testing Machine | Kali Linux |
| Ubuntu IP Address | 192.168.223.131 |
| Kali IP Address | 192.168.223.129 |
| Firewall | UFW |
| Scanning Tool | Nmap |

## 3. Initial Firewall Assessment

The initial UFW status was checked with:

```bash
sudo ufw status verbose
```

Result: `Status: inactive`.

SSH was confirmed to be listening on TCP port 22, on IPv4 and IPv6 interfaces, using:

```bash
sudo ss -tlnp | grep ':22'
```

## 4. Firewall Configuration

SSH was allowed:

```bash
sudo ufw allow 22/tcp
```

The default incoming policy was set to deny:

```bash
sudo ufw default deny incoming
```

The default outgoing policy was set to allow:

```bash
sudo ufw default allow outgoing
```

UFW was enabled:

```bash
sudo ufw enable
```

## 5. Firewall Verification

The active configuration was checked with `sudo ufw status verbose` and reported:

- Status: Active
- Logging: On (low)
- Default incoming policy: Deny
- Default outgoing policy: Allow
- SSH TCP port 22: Allowed, with IPv4 and IPv6 rules present

## 6. Connectivity Testing

### ICMP Ping

From Kali Linux:

```bash
ping -c 4 192.168.223.131
```

Observed result: 4 packets transmitted, 4 received, 0% packet loss; average response time 1.207 ms. Kali could reach Ubuntu after UFW was enabled.

### SSH

From Kali Linux:

```bash
ssh raymond-favour@192.168.223.131
```

The SSH connection succeeded, confirming that TCP port 22 was allowed through the firewall.

## 7. Nmap Scans

### General Service Scan

```bash
nmap -sV 192.168.223.131
```

Observed results: host up; TCP port 22 open (SSH, OpenSSH 10.2p1 Ubuntu 2ubuntu3.6); the other 999 scanned TCP ports filtered.

### Targeted Port Scan

```bash
nmap -sV -p 22,80,443,631 192.168.223.131
```

| Port | Service | State |
|---|---|---|
| 22/tcp | SSH | Open |
| 80/tcp | HTTP | Filtered |
| 443/tcp | HTTPS | Filtered |
| 631/tcp | IPP/CUPS | Filtered |

SSH remained accessible, while the other tested ports were filtered.

## 8. Security Findings

1. UFW was initially inactive.
2. A default-deny incoming policy restricts unsolicited inbound connections.
3. Outgoing traffic is allowed for normal network operation.
4. SSH was explicitly allowed for remote administration.
5. Nmap found SSH accessible and the other tested ports filtered.
6. Kali retained ping and SSH connectivity after firewall activation.

## 9. Skills Practised

- Linux firewall administration and UFW configuration
- Default firewall policies and TCP port rules
- SSH connectivity verification
- ICMP network testing
- Nmap service detection and port scanning
- Basic network security hardening and technical documentation

## 10. Conclusion

UFW was enabled on Ubuntu with a default-deny incoming policy and default-allow outgoing policy. SSH was explicitly permitted. The recorded connectivity checks and Nmap scans showed that SSH access remained available while the other tested ports were filtered.
