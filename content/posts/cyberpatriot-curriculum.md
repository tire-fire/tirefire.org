+++
title = "Cyber Patriot Curriculum"
date = "2023-06-20"
author = "tirefire"
description = "Cyber Patriot Proposed Curriculum"
+++

## Cyber Patriot Proposed Curriculum

## Official Training Modules

### Nine Units worth of traning materials on https://www.uscyberpatriot.org

- Introduction to CyberPatriot and Cybersecurity
- Introduction to Online Safety
- Cyber Ethics
- Principles of Cybersecurity
- Computer Basics and Virtualization
- Microsoft Windows Basic
- Microsoft Windows Security Tools
- Microsoft Windows Security Configuration
- Introduction to Linux and Ubuntu


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
    - RDP, SMB, LDAP, Kerberos, DNS, Print, WinRM, etc...

#### Operating System Hardening and Updates

- Windows Update Management
  - Applying critical updates, security patches, and feature updates to the Windows operating system
- Configuring security options, group policies, and security templates to harden the Windows environment
- Event Log audit policies, monitoring logs, and analyzing event data for security incidents
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
  - ssh, web servers, ftp servers, etc...

#### Operating System Hardening and Updates

- OS Patching and Updates
  - Applying security patches and updates to the Linux operating system
- Securing System Settings
  - Configuring security options, system services, and secure shell (SSH) settings
- System Monitoring and Logging
  - Implementing audit policies, monitoring logs, and detecting suspicious activities
- Firewall Configuration and Rules
  - iptables, ufw
- Scheduled Tasks
  - cron, at, systemd
- Init system
  - systemd, sysvinit
- Prohibited Files and Software Handling
  - File permissions


## Example servers and services
- Apache
- IIS
- nginx
- PostgreSQL
- MySQL
- SSH
- MSSQL
- VNC
- LAMP Server (Linux Apache Mysql PHP) and variations
- Wordpress
- SMB
- FTP
- DNS
- Active Directory / Samba
- Mail


## Frameworks and Databases to use for resources
- MITRE https://attack.mitre.org/
- STIGs https://public.cyber.mil/stigs/
- CIS Benchmarks https://www.cisecurity.org/cis-benchmarks
- HackTricks https://book.hacktricks.xyz
- CyberChef https://gchq.github.io/CyberChef/


## Networking
- NetAcad
