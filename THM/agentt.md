
# TryHackme Agent T Write-up

Below is my write-up for the room [Agent T](https://tryhackme.com/room/agentt).

We start the machine and let us run a `nmap` scan to see what we can find.

```bash
└─$ sudo nmap -sC -sV 10.10.245.251
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-07 22:00 AEST
Stats: 0:00:12 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 0.00% done
Nmap scan report for 10.10.245.251
Host is up (0.30s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard

```
We can only see one port open. This just lands us on an SB Admin 2 page.

![SBAdminexample](/THM/Images/AgentT/AgentTdashboard.png)

Looking through the HTML code and running a gobuster gives us nothing useful.

Let's see what we can capture through Burp Suite. The only thing we can see that really interests us is the PHP build running on the server `PHP 8.1.0 dev`.


![BurpsuiteCapture](/THM/Images/AgentT/AgentTBurpsuiteCapture.png)

If we search through the exploit database, we can find the vulnerability. Exploit by `flast101` - [Exploit DB webpage](https://www.exploit-db.com/exploits/49933). The specific version was released with a back door on March 28th 2021, but the backdoor was quickly discovered and patched up. An attacker can execute arbitrary code by sending the commands on the User-Agent header.

There are a few good examples of exploits around, but what they all demonstrate is that sending 

```bash
headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0",
            "User-Agentt": "zerodiumsystem('" + cmd + "');" # Here is where our commands will be sent
            }

```
The server will run those commands. Using the Python script from `flast101`. 

![Flag](/THM/Images/AgentT/AgentTPythonRCE.png)

We are able to see that the vulnerability is high risk as we can send commands to the server as root. 
