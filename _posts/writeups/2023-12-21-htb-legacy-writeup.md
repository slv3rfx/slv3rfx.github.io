---
title: "Hack The Box: Legacy Write-Up"
date: 2023-12-21 21:43
categories: [Write-ups]
tags: [htb, lab, windows, metasploit, nmap, smb]
img_path: /htb/machines/legacy/
---

Legacy is an easy-rated Windows machine. 

## Enumeration

Let's start by using a basic nmap scan.

```console
$ nmap -sC -sV 10.10.10.4 -oA scans/basic_scan
# Nmap 7.94SVN scan initiated Thu Dec 21 15:14:39 2023 as: nmap -sC -sV -oA scans/basic_scan 10.10.10.4
Nmap scan report for 10.10.10.4
Host is up (0.22s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:33:2f (VMware)
|_clock-skew: mean: 5d00h57m39s, deviation: 1h24m50s, median: 4d23h57m39s
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2023-12-27T00:12:54+02:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 21 15:15:25 2023 -- 1 IP address (1 host up) scanned in 46.25 seconds
```

We can see that this host is very likely running Windows XP and SMBv1. This makes it very vulnerable to many known exploits. 

## Exploitation

I decided to open Metasploit and choose the module for the well-known [EternalChampion](https://www.exploit-db.com/exploits/43970) exploit.

![Metasploit Options](/msf-options.png)

After executing it without authentication, I successfully got a reverse shell as Administrator back.

![Metasploit Reverse Shell](/msf-reverse-shell.png)

The flags are in the following directories:
- User: C:\Documents and Settings\john\Desktop
- Root: C:\Documents and Settings\Administrator\Desktop
