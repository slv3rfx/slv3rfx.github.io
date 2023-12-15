---
title: "HackTheBox: Jerry Write-Up"
date: 2023-11-12 12:55
categories: [HackTheBox, Machines]
tags: [windows, metasploit, tomcat, nmap]
---

Jerry is an easy-rated Windows machine on HTB that can teach you some basics of proper enumeration, searching for exploits on the Internet, and brute-forcing valid credentials for protected pages.

## Enumeration

Let’s start by enumerating the target with a standard nmap scan.

```console
$ nmap -oA scans/nmap_initial -Pn 10.10.10.95
```

![Initial Nmap Scan](/htb/machines/jerry/nmap-scan.png)

We can see that there is a web server running on port 8080. Let’s do another scan for this port with the default scripts and service detection.

```console
$ nmap -oA scans/nmap_scripts -sV -sC -p 8080 -Pn 10.10.10.95
```

![Nmap Scripts](/htb/machines/jerry/nmap-scripts.png)

Through the http title, we find out that it’s an Apache Tomcat server with the version 7.0.88.

When we try to visit the website, we are presented with Tomcat’s default page saying that it was installed successfully. There, we can see two buttons that look interesting: “Manager App” and “Host Manager”. But we need credentials for both of them.

![Tomcat Default Page](/htb/machines/jerry/tomcat-default-page.png)

When searching for exploits for this version of Tomcat, we also come across this HackTricks article. It describes how we can achieve Remote Code Execution (RCE) by uploading a malicious .war file. However, there is a catch: We first need to be able to access the Tomcat Web Application Manager. As we saw in the last step, we first have to find some valid credentials.

## Finding Valid Credentials

There are several possible ways we can go about finding the credentials we need. You could search on the Internet for standard Tomcat usernames and passwords. When you do this, you will find lists, like this or this. Trying them one by one manually would be one option. Another option would be to write a script yourself in your preferred language, iterating over the list and testing the validity of each pair. You could also use a tool such as Hydra.

However, in our case, there is also a Metasploit module available so we’ll use that.

![Metasploit Tomcat Scanner](/htb/machines/jerry/metasploit-tomcat-scanner.png)

When we run this scanner, we will get one pair of valid credentials: **tomcat:s3cret**

## Exploitation

With the newly acquired credentials, we can now exploit the above-mentioned RCE vulnerability. In order to do this, you could create your own .war file and upload it manually. The official write-up explains how you can do that.

Since there is a Metasploit module available for this step as well, we will take the easy route.

![Metasploit Exploit](/htb/machines/jerry/metasploit-exploit.png)

> Set the LHOST option to the IP address of your tun0 interface
{: .prompt-info}

After we run this, we will get a Meterpreter shell on the target machine as the user JERRY.

Both flags can be found in the _C:\Users\Administrator\Desktop\flags_ directory.

![Flags](/htb/machines/jerry/flags.png)