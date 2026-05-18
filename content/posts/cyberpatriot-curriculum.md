+++
title = "Cyber Patriot Curriculum"
date = "2025-12-26"
author = "tirefire"
description = "Cyber Patriot Curriculum"
+++

## Cyber Patriot Curriculum

## Official Training Modules

### Nine Units worth of training materials on https://www.uscyberpatriot.org/competition/training-materials/training-modules

- Introduction to CyberPatriot and Cybersecurity
- Introduction to Online Safety
- Cyber Ethics
- Principles of Cybersecurity
- Computer Basics and Virtualization
- Microsoft Windows Basic
- Microsoft Windows Security Tools
- Microsoft Windows Security Configuration
- Introduction to Linux and Ubuntu

### Forensics Questions

Every round includes forensics questions that require:
- Locating files/directories based on clues
- Identifying users from logs or configurations
- Reading service banners and configuration values
- Hash identification (MD5, SHA)
- Analyzing system state to answer scenario questions

Common artifact locations:
- **Windows**
  - Event Log IDs: 4624 (logon), 4625 (failed logon), 4720 (account created), 4732 (added to group), 7045 (service install), 4688 (process create)
  - Registry run keys: `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`, `RunOnce`, `HKCU\...\Run`
  - Scheduled Tasks: `C:\Windows\System32\Tasks\`
  - Startup folders: `%AppData%\Microsoft\Windows\Start Menu\Programs\Startup`
  - Prefetch: `C:\Windows\Prefetch\`
  - PowerShell history: `%AppData%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
- **Linux**
  - Shell history: `~/.bash_history`, `~/.zsh_history`
  - SSH: `~/.ssh/authorized_keys`, `/etc/ssh/sshd_config`
  - Auth logs: `/var/log/auth.log`, `/var/log/secure`, `/var/log/wtmp`, `/var/log/btmp`, `last`, `lastb`
  - Cron: `/etc/crontab`, `/etc/cron.*`, `/var/spool/cron/`
  - Systemd units: `/etc/systemd/system/`, `~/.config/systemd/user/`
  - Persistence: `/etc/rc.local`, `/etc/profile.d/`, `~/.bashrc`, `~/.profile`


### Windows

#### Account Security and Management

- Password Policy
  - Defining password complexity, length, expiration, history, and hashing algorithms
  - Configuring account lockout duration, threshold, administrative privileges, and reset procedures
- User Account Management
  - Creating, modifying, and disabling user accounts with appropriate access levels
  - Managing user accounts, group memberships, and access privileges to ensure appropriate access control
  - Updating passwords

#### Application Security and Updates

- Application Updates
  - Updates can be unique to each application
  - Windows Updates can have some application updates
  - Automatic updates when available
  - Winget / Nuget / Ninite / Chocolatey
  - About :: Help :: Check for updates
- Software Installation Control
  - Software installation permissions and preventing unauthorized applications using group policy
- Application configuration and hardening
  - Enable secure settings for applications such as Firefox or Internet Explorer and audit addons
- Service Hardening and Disabling
  - Evaluating and disabling unnecessary or insecure services for enhanced security
  - Remote access services hardened or disabled
    - RDP (Network Level Authentication), Remote Assistance
    - SMB (disable v1, signing, encryption, share permissions vs NTFS permissions, hidden shares, null sessions, anonymous access)
    - LDAP, Kerberos, DNS, Print, WinRM, etc...

#### Operating System Hardening and Updates

- Windows Update Management
  - Applying critical updates, security patches, and feature updates to the Windows operating system
- Configuring security options, group policies, and security templates to harden the Windows environment
  - Anonymous enumeration of SAM accounts
  - Blank password restrictions (limit to console only)
  - CTRL+ALT+DEL logon requirement
  - SMB signing requirements
  - Network security options and privilege elevation
- Event Log audit policies, monitoring logs, and analyzing event data for security incidents
  - Sysmon from Sysinternals
- PowerShell logging and hardening
  - Script block logging
  - Module logging
  - Transcription
  - Constrained Language Mode
  - Execution policy
- Configuring firewall rules to allow critical services
- Antivirus and Endpoint Protection
- BitLocker Encryption
- Software/malware Detection and Removal
  - Identifying and removing backdoors, keyloggers, rootkits, and other malware using antivirus tools and security software
  - Using Sysinternals procmon/procexp/autoruns for detection
- Application allowlisting
  - AppLocker (rules, enforcement, audit mode)
  - Windows Defender Application Control (WDAC)
- Credential protection
  - LAPS (Local Administrator Password Solution)
  - Credential Guard
  - Protected Users group
  - LSA Protection
- Defender Attack Surface Reduction (ASR) rules
- Prohibited Files and Software Handling
  - Detecting and addressing prohibited files, unauthorized software, and potential security risks
  - Alternate data streams
  - icacls
  - Identifying and removing unwanted games, scareware, adware, potentially unwanted programs (PUPs), and hacking tools
- Scheduled Tasks
- File sharing
- Local Group Policy Editor and LGPO templates


### Linux

#### Account Security and Management

- Password Policy
  - PAM configuration
    - pam_pwquality / pam_cracklib
    - pam_tally2 / pam_faillock
    - pam_unix
    - /etc/pam.d/ stack ordering
  - Setting password length, age, complexity requirements, and hashing algorithms
  - Account Lockout Policy
  - Configuring lockout duration, threshold, administrators' privileges, and reset procedures
- User Account Management
  - Creating, modifying, and disabling user accounts with appropriate access levels
  - Updating passwords
  - /etc/passwd
  - /etc/shadow
  - /etc/group
- User Rights and Permissions
  - sudo
    - visudo
    - /etc/sudoers and /etc/sudoers.d/
    - NOPASSWD entries
    - wheel/sudo group membership
- SSH access controls
  - Key-based authentication (authorized_keys, key types, permissions)
  - PermitRootLogin, PasswordAuthentication, PermitEmptyPasswords
  - AllowUsers / AllowGroups / DenyUsers
  - fail2ban (jail configuration, ban thresholds)

#### Application Security and Updates

- Application updates
  - package manager (apt, yum, dnf, apk, etc...)
    - packages (dpkg, rpm, etc...)
  - snap, flatpak, appimage
- Repository Management
- Updating configurations for critical services
  - ssh (disable root login, key-based auth, protocol version)
  - web servers (Apache, nginx)
  - ftp servers (vsftpd SSL/TLS, directory permissions)
- User applications
  - Enable secure settings for user applications such as Firefox and audit addons

#### Operating System Hardening and Updates

- OS Patching and Updates
  - Applying security patches and updates to the Linux operating system
- Securing System Settings
  - Configuring security options, system services
  - Display managers (lightdm, gdm, sddm)
  - X display, Wayland
  - dbus
  - polkit
  - systemd logind
  - File system mount options
  - Guest account management
- System Monitoring and Logging
  - Implementing audit policies, monitoring logs, and detecting suspicious activities
  - syslog
  - auditd
- Firewall Configuration and Rules
  - iptables, ufw, nftables, firewalld
- Kernel hardening
  - IPv4 TCP SYN cookies
  - sysctl security parameters
- Mandatory Access Control
  - AppArmor (profiles, enforce vs complain mode, aa-status)
  - SELinux (contexts, modes, semanage, restorecon)
- Rootkit and malware detection
  - chkrootkit
  - rkhunter
  - Unexpected SUID/SGID binaries
  - Suspicious cron entries and systemd units
  - LD_PRELOAD and /etc/ld.so.preload abuse
- Scheduled Tasks
  - cron, at, systemd
- Init system
  - systemd, sysvinit
- Prohibited Files and Software Handling
  - File permissions
  - Extended attributes
  - SUID/SGID auditing (`find / -perm -4000`, `find / -perm -2000`)
  - World-writable files and directories
  - Files with no owner (`find / -nouser -o -nogroup`)


## Example servers and services
- Web servers: Apache, IIS, nginx, Caddy
- Databases: PostgreSQL, MySQL, MongoDB, MSSQL
- Web stacks/CMS: LAMP, XAMPP, Wordpress, Joomla
- File sharing: SMB, FTP (vsftpd, Filezilla, IIS FTP), NFS
- Directory services: Active Directory / Samba, AD Certificate Services, LDAP (OpenLDAP), Kerberos
- Remote access: SSH, VNC, RDP, WinRM, VPN (OpenVPN, WireGuard, StrongSwan, RRAS)
- Proxy: Squid, HAProxy, Varnish, Traefik
- DNS: bind9, Microsoft DNS, dnsmasq, Unbound
- Mail: Postfix, Dovecot, sendmail, Microsoft Exchange, MailEnable, Roundcube
- Monitoring: ELK stack
- NTP


## Frameworks and Databases to use for resources
- MITRE https://attack.mitre.org/
- STIGs https://public.cyber.mil/stigs/
- CIS Benchmarks https://www.cisecurity.org/cis-benchmarks
- HackTricks https://book.hacktricks.xyz
- CyberChef https://gchq.github.io/CyberChef/


## Cisco
- NetAcad

### Core Networking
- Basic device configuration
- Password encryption
- Line vty configuration
- VLANs and trunking
- Routing (static, OSPF)
- EtherChannel
- Port security
- Spanning Tree Protocol (STP)
- ACLs

### Critical Service Configuration
- NTP
- DNS
- DHCP
- AAA
- VPN
- SSH
- FTP
- SNMP

### Security Hardening
- Limit device access
- Disable unused ports and interfaces
  - shutdown
- Interface hardening
  - no ip redirects
  - no ip proxy-arp
  - no ip directed-broadcast
- Disabling unnecessary services
  - no ip http server
  - no service finger
- BPDU Guard
- DHCP snooping
- Dynamic ARP Inspection (DAI)
- IP Source Guard (IPSG)
- MAC address sticky
- Banner
- Rate Limiting
- IP Options
- Logging
- Routing protocol security

### Wireless
- WPA2, WPA3
- SSID configuration

### Troubleshooting
- Incorrect IP addresses
- Interface descriptions
- Duplex/speed mismatches
- Trunking problems


## Contributors

Thanks to the following for proposed additions:

- **StageKing** — Cisco section structure
- **Henry** — SMB hardening expansion, PowerShell script block logging, AppArmor, PAM, rootkit detection; Cisco additions
