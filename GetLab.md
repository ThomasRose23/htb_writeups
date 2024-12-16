# Writeup

# Enumeration
---
### Nmap
#

sudo nmap -v -sS -sV -sC -T4 -p- -oA getbash-tcp-fullportscan 10.129.229.69

```bash
# Nmap 7.94SVN scan initiated Mon Dec 16 11:33:36 2024 as: nmap -v -sS -sV -sC -T4 -p- -oA getbash-tcp-fullportscan 10.129.229.69
Nmap scan report for 10.129.229.69
Host is up (0.050s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 f3:e8:cf:2e:39:5f:c9:c8:f1:e8:97:a8:04:6a:28:c2 (RSA)
|   256 9b:8a:64:29:f0:b8:c3:c0:4c:55:f0:78:27:ce:c5:86 (ECDSA)
|_  256 c8:2b:18:14:bd:77:2b:0c:68:f6:c3:cb:92:46:34:66 (ED25519)
80/tcp   open  http    nginx
|_http-favicon: Unknown favicon MD5: 66F9A1C3F2CFD0DF1B570990E86D3095
|_http-trane-info: Problem with XML parsing of /evox/about
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 57 disallowed entries (15 shown)
| / /autocomplete/users /autocomplete/projects /search 
| /admin /profile /dashboard /users /api/v* /help /s/ /-/profile 
|_/-/ide/ /-/experiment /*/new
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.129.229.69/users/sign_in
8060/tcp open  http    nginx 1.20.2
|_http-title: 404 Not Found
|_http-server-header: nginx/1.20.2
| http-methods: 
|_  Supported Methods: GET HEAD POST
9094/tcp open  unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Dec 16 11:34:12 2024 -- 1 IP address (1 host up) scanned in 36.71 seconds
```
