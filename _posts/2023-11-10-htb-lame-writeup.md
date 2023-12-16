---
title: "Hack The Box: Lame Write-Up"
date: 2023-11-10 15:35
categories: [Hack The Box, Machines]
tags: [windows, smb, metasploit, nmap]
img_path: /htb/machines/lame/
---

As I am still new in the world of cybersecurity and penetration testing, I want to document my progress on my learning. I will be starting with this write-up of the Lame machine on HackTheBox, showing my approach and thought process while pwning it.

First, I started with a typical nmap scan. Since nmap’s ping packets seem to get blocked, I had to add the Pn switch to tell nmap to scan the ports regardless.

```console
$ nmap --open -sC -Pn -oA scans/nmap_initial {TARGET_IP}
Nmap scan report for 10.10.10.3
Host is up (0.050s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.16.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 600fcfe1c05f6a74d69024fac4d56ccd (DSA)
|_  2048 5656240f211ddea72bae61b1243de8f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2023-11-01T13:02:03-04:00
|_clock-skew: mean: 2h00m22s, deviation: 2h49m43s, median: 21s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Wed Nov  1 13:02:21 2023 -- 1 IP address (1 host up) scanned in 52.44 seconds
```

According to the scan results, there are four ports open.

The ports are associated with the services FTP, SSH, and SMB. As it seems that anonymous login is enabled on FTP, I tried connecting to it. Unfortunately, I quickly found out that it was empty.

Next, I had a look at the shares exposed by SMB.

![SMB Shares](smb-shares.png)

The _tmp_ share looked interesting so I connected to it. But I did not find anything of interest.

![SMB tmp directories](smb-tmp-directories.png)

When I took another look at the output of the share listing, I noticed that Samba 3.0.20 is being used. Then, I searched the Internet for vulnerabilities for this particular version of Samba. I discovered that it has the CVE number [CVE-2007-2447](https://nvd.nist.gov/vuln/detail/CVE-2007-2447) as well as a [Metasploit module](https://www.exploit-db.com/exploits/16320) which gives us an immediate root shell.

Afterwards, I quickly started Metasploit and searched for Samba and set the necessary options.

![Metasploit Options](metasploit-options.png)

> You still need to set the LHOST option to the IP address of your tun0 interface.
{: .prompt-info}

After running the exploit, you have access to a reverse shell as the root user.

![Metasploit Reverse Shell](metasploit-reverse-shell.png)

You can find the two flags at /home/makis/user.txt and /root/root.txt, respectively.
