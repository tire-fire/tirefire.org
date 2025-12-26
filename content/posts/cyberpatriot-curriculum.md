+++
title = "Cyber Patriot Curriculum"
date = "2025-12-26"
author = "tirefire"
description = "Cyber Patriot Proposed Curriculum"
+++

## Cyber Patriot Proposed Curriculum

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
    - SMB (disable v1, signing, share permissions vs NTFS permissions, hidden shares)
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
- Configuring firewall rules to allow critical services
- Antivirus and Endpoint Protection
- BitLocker Encryption
- Software/malware Detection and Removal
  - Identifying and removing backdoors, keyloggers, rootkits, and other malware using antivirus tools and security software
  - Using Sysinternals procmon/procexp/autoruns for detection
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
  - PAM config
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
- Scheduled Tasks
  - cron, at, systemd
- Init system
  - systemd, sysvinit
- Prohibited Files and Software Handling
  - File permissions
  - Extended attributes


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
- Minecraft Server


## Frameworks and Databases to use for resources
- MITRE https://attack.mitre.org/
- STIGs https://public.cyber.mil/stigs/
- CIS Benchmarks https://www.cisecurity.org/cis-benchmarks
- HackTricks https://book.hacktricks.xyz
- CyberChef https://gchq.github.io/CyberChef/


## Networking
- NetAcad
- Limit device access
- Password encryption
- Interface hardening
  - Disable unused interfaces
    - shutdown
  - no ip redirects
  - no ip proxy-arp
  - no ip directed-broadcast
- Routing protocol security
- Disabling unnecessary services
  - no ip http server
  - no service finger
  - no service ...
- Banner
- Rate Limiting
- IP Options
- Logging
- NTP
