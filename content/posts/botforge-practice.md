+++
title = "BotForge Practice Image"
date = "2025-12-30"
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
Seven questions on the Desktop (Forensics1.txt - Forensics7.txt). They're breadcrumbs - answer them and you'll stumble into most of the findings. Each references a MITRE ATT&CK technique ID if you want to dig deeper.

## Tips
- Check what's running, what's listening, what's scheduled
- Read configs carefully - the devil is in the details
- Git remembers things people wish it would forget
- Not everything malicious looks malicious
- The logs tell a story if you know where to look

## Download
The link is here: https://ln5.sync.com/dl/a7d6d1f80#6mqzmf43-cg46ipx9-2cfvjtj4-p364qeqt \
md5sum: `5276b58c0cdfba014b53c2a05ae55245  botforge-mint22.zip`

## Requirements
- VMware Workstation, Player, or Fusion
- 4GB RAM minimum (8GB recommended)
- 20GB disk space

## Errata
None yet. Report issues via the contact page.

## Writeup
Coming eventually.
