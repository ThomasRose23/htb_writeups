# BountyHunter

# Enumeration
---
### Nmap
#

TCP scan of all ports, including default scripts.

```bash
# Command
nmap -v -sS -sV -sC -T4 -p- -oA bountyhunter-full-tcp 10.129.95.166

# Results
Nmap scan report for 10.129.95.166
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Bounty Hunters
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Dec 23 11:53:23 2024 -- 1 IP address (1 host up) scanned in 27.94 seconds
```

# HTTTP (Port 80)

Port 80 hosts a Bug Bounty submission page, browsing around there are a few stand out pages, such as a contact form which could be used for injection, however more interestingly a portal used for submitting bugs to the system. 

![image](https://github.com/user-attachments/assets/b08ce87c-dddc-4237-b0e3-6b0d70e71c5c)

A message is displayed on submission stating the DB isn't setup, however, the results are returned to the page. Not taking the comment at face value, I try some basic injection attacks including SQL but nothing works.

Some other paths were discovered using ffuf:

  Command line : `ffuf -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://bountyhunter/assets/FUZZ -t 2 -v -o ffuf-forced-browsing-directories -of md`
  Time: 2025-01-02T10:46:40-05:00

  | FUZZ | URL | Redirectlocation | Position | Status Code | Content Length | Content Words | Content Lines | Content Type | Duration | ResultFile | ScraperData | Ffufhash
  | :- | :-- | :--------------- | :---- | :------- | :---------- | :------------- | :------------ | :--------- | :----------- | :------------ | :-------- |
 | http://bountyhunter/assets/ |  | 14 | 403 | 277 | 20 | 10 | text/html; charset=iso-8859-1 | 21.886996ms |  |  | 58677e
  | # | http://bountyhunter/assets/# |  | 13 | 403 | 277 | 20 | 10 | text/html; charset=iso-8859-1 | 22.443474ms |  |  | 58677d
  | img | http://bountyhunter/assets/img | http://bountyhunter/assets/img/ | 39 | 301 | 317 | 20 | 10 | text/html; charset=iso-8859-1 | 22.871146ms |  |  | 5867727
  |  | http://bountyhunter/assets/ |  | 45647 | 403 | 277 | 20 | 10 | text/html; charset=iso-8859-1 | 22.757757ms |  |  | 58677b24f
