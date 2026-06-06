# Common Ports Cheat Sheet

## Quick Reference

| Port | Protocol | Service | Notes |
|---|---|---|---|
| 20 | TCP | FTP Data | File transfer data channel |
| 21 | TCP | FTP Control | File transfer — check for anonymous login |
| 22 | TCP | SSH | Secure shell — check for weak credentials |
| 23 | TCP | Telnet | Unencrypted remote access — legacy, rarely seen |
| 25 | TCP | SMTP | Email sending — check for open relay |
| 53 | TCP/UDP | DNS | UDP for queries, TCP for zone transfers |
| 67/68 | UDP | DHCP | Network config assignment |
| 69 | UDP | TFTP | Trivial FTP — no authentication |
| 80 | TCP | HTTP | Web traffic — always enumerate |
| 88 | TCP | Kerberos | AD authentication — signals Domain Controller nearby |
| 110 | TCP | POP3 | Email retrieval |
| 111 | TCP/UDP | RPCbind | NFS prerequisite — check for NFS shares |
| 119 | TCP | NNTP | Usenet — rarely seen |
| 123 | UDP | NTP | Time sync |
| 135 | TCP | MSRPC | Windows RPC — common on Windows targets |
| 137-139 | TCP/UDP | NetBIOS | Windows name resolution and file sharing |
| 143 | TCP | IMAP | Email retrieval |
| 161/162 | UDP | SNMP | Network device management — check for default community strings |
| 389 | TCP | LDAP | Directory services — signals AD environment |
| 443 | TCP | HTTPS | Encrypted web — enumerate same as HTTP |
| 445 | TCP | SMB | Windows file sharing — high value target, check for EternalBlue |
| 465 | TCP | SMTPS | Encrypted email sending |
| 500 | UDP | IKE | VPN key exchange |
| 512 | TCP | rexec | Remote execution — legacy Unix |
| 513 | TCP | rlogin | Remote login — legacy Unix, no encryption |
| 514 | TCP/UDP | Syslog / rsh | Log data or remote shell — legacy |
| 587 | TCP | SMTP Submission | Modern email sending with auth |
| 631 | TCP | IPP | Printing — sometimes exposed |
| 636 | TCP | LDAPS | Encrypted LDAP |
| 873 | TCP | Rsync | File sync — sometimes allows anonymous access |
| 902 | TCP | VMware | VMware ESXi management |
| 993 | TCP | IMAPS | Encrypted IMAP |
| 995 | TCP | POP3S | Encrypted POP3 |
| 1080 | TCP | SOCKS | Proxy — check for open proxy |
| 1433 | TCP | MSSQL | Microsoft SQL Server |
| 1521 | TCP | Oracle DB | Oracle database |
| 2049 | TCP/UDP | NFS | Network file shares — check for world-readable mounts |
| 2121 | TCP | FTP Alt | Alternate FTP port |
| 3306 | TCP | MySQL | MySQL database |
| 3389 | TCP | RDP | Windows Remote Desktop — check for BlueKeep, weak creds |
| 4444 | TCP | Metasploit | Default Metasploit listener port |
| 5432 | TCP | PostgreSQL | PostgreSQL database |
| 5900 | TCP | VNC | Remote desktop — check for no/weak password |
| 5985 | TCP | WinRM HTTP | Windows Remote Management — PowerShell remoting |
| 5986 | TCP | WinRM HTTPS | Encrypted WinRM |
| 6379 | TCP | Redis | In-memory database — often no auth by default |
| 6667 | TCP | IRC | Internet Relay Chat — UnrealIRCd backdoor (classic box) |
| 8080 | TCP | HTTP Alt | Common alternate web port — always enumerate |
| 8443 | TCP | HTTPS Alt | Alternate HTTPS |
| 8888 | TCP | HTTP Alt | Another common alternate web port |
| 9200 | TCP | Elasticsearch | Search engine — often no auth |
| 27017 | TCP | MongoDB | NoSQL database — often no auth by default |

---

## High Value Targets for Pentesting

| Port | Why It Matters |
|---|---|
| 21 | Anonymous FTP login, cleartext credentials |
| 22 | Brute force, key misconfigurations |
| 23 | Cleartext credentials |
| 80/443 | Web app vulnerabilities, directory enumeration |
| 139/445 | SMB — EternalBlue, pass-the-hash, enum shares |
| 1433/3306/5432 | Database access, credential dumping |
| 3389 | RDP brute force, BlueKeep |
| 5900 | VNC with no password |
| 6379 | Redis with no auth |

---

## Active Directory Giveaway Ports
When you see these together, you're probably looking at a Domain Controller:

- 88 (Kerberos)
- 389 (LDAP)
- 445 (SMB)
- 636 (LDAPS)
- 3268/3269 (Global Catalog)

---

## Notes
- Default nmap scans the top 1000 most common ports — not ports 1-1000
- Use `-p-` to scan all 65535 ports
- UDP services (DNS, SNMP, DHCP) are often missed — use `-sU` to catch them
- Services don't always run on their default port — always check with `-sV`
