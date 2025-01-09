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

The version of Apache Spark running can be seen in the top left next to the logo, the version in use is 3.0.3. 
