+++
title = "Fedora Practice Image"
date = "2022-11-26"
author = "tirefire"
cover = "img/ecorp.jpg"
description = "Fedora 37 practice image for securing and hardening an operating system"
+++

## What is it?
This is a practice image for securing and hardening an operating system. This practice image is based on Fedora 37 and should help with learning Fedora as well as other Linux internal systems.

## Why?
To give some challenges which introduce technologies which students might not be familiar with.

## Theme
The theme is *Mr. Robot*, but really it's just based on a short scene from Season 3 Episode 5 titled *eps3.4_runtime-error.r00*, and only loosely takes influence. Knowing Mr. Robot won't give any advantage, and doing this image won't spoil any of Mr. Robot.

## Getting started
The image should automatically log in. The username and passwords are in the readme file. If you lock yourself out, please re-extract and start over.

## Forensics Questions
There are a lot more forensics questions than most images. I felt like students don't get enough direction, so virtually every finding in the image gets a hint from a forensics question. As you answer the questions, you'll find references to a T number, these correspond to the data found in the MITRE ATT&CK framework at https://attack.mitre.org/matrices/.

## Download
The link is here https://ln5.sync.com/dl/3e4ff83c0/u2wyn5b9-yuthhkgw-dqhghbr4-qme8tqjx \
with a mirror here https://drive.google.com/file/d/1QQYUdp1Qu1-0NXf7VHyiIMl2QY4X_izp/view \
md5sum: `5ea0ff7823ff8bd8578a987d80a99436  mr.robot.sh.zip`

## Errata
Here are mistakes that are in the image:
- Forensics Question 5 is incorrect. It should be the shell, not the user, and is not related to Fred.
- Forensics Question 9: a hint to use `about:debugging` in Firefox
- Forensics Question 12: a hint to use `rpm -V`
- Forensics Question 13 should refer to the user's password-less sudo access. Additionally, related findings are mistakenly not scored.
- Forensics Question 14 just has information about the password so that you don't try to brute force it. You won't be able to find it on the file system, until you "unhide" it.
