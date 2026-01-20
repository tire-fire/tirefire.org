+++
title = "BotForge Practice Image"
date = "2026-01-17"
author = "tirefire"
cover = "img/tirefire.gif"
description = "Linux Mint 22 practice image for securing and hardening an operating system"
+++

## What is it?
A practice image for learning incident response and system hardening. Based on Linux Mint 22, it should help with learning Mint as well as other Debian-based systems.

## Why?
To practice incident response on a realistic scenario. The findings connect to each other - the attacker had a motive, a method, and made mistakes you can trace.

## Theme
*BotForge* is a small Discord bot hosting company. Think shared hosting, but for bots - customers upload code, BotForge runs it on shared infrastructure.

Three weeks ago, a customer called "vex" got terminated for running phishing bots. They didn't take it well. Now the sysadmin is seeing strange network traffic, mystery processes at 3am, and a customer complaining about a leaked bot token.

You're cleaning up the mess.

## Difficulty
Intermediate. If you've done a few practice images before, you should be comfortable here. If this is your first one, expect to learn a lot (and struggle a bit).

## Getting started
The image auto-logs in as mford. Read the README on the Desktop - it has the scenario details and lists who should (and shouldn't) be on the system.

**Take a snapshot before you start.** You'll probably break something.

## Forensics Questions
Seven questions on the Desktop (Forensics1.txt - Forensics7.txt). They're breadcrumbs - answer them and you'll stumble into most of the findings. Each references a MITRE ATT&CK technique ID (like T1053) - search for it at [attack.mitre.org](https://attack.mitre.org) to learn what the technique is and how attackers use it.

When you fix a vulnerability, the scoring report shows a hint pointing you toward related findings.

## Tips
- Check what's running, what's listening, what's scheduled
- Read configs carefully - the devil is in the details
- Git remembers things people wish it would forget
- Not everything malicious looks malicious
- The logs tell a story if you know where to look

## Download
https://downloads.tirefire.org/botforge-practice-v2.ova

md5sum: `c5ef4ab19fd6cd9f79139ba6db2b65b9`

## Credentials
- **Username:** mford
- **Password:** mford123

## Requirements
- VMware Workstation, Player, or Fusion
- 4GB RAM minimum (8GB recommended)
- 20GB disk space

## Errata
- **Nginx directory traversal:** The check expects you to remove the alias line, but the real fix is adding a trailing slash to the `location` directive. This one won't score correctly - skip it.
- **MariaDB root password:** The check expects "Access denied" when connecting via TCP, but if MariaDB is socket-only you'll get "Can't connect" instead. May not score even when properly secured.

## Writeup
https://github.com/christopher-gholmieh/BotForge-Writeup
