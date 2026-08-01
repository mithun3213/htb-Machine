
Basic recon:

```
oxdf@hacky$ sudo nmap -p- -vvv --min-rate 10000 10.129.17.88
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-29 19:18 UTC
... [snip]

Nmap scan report for 10.129.17.88
Host is up, received reset ttl 63 (0.024s latency).
Scanned at 2026-03-29 19:18:57 UTC for 7s

Not shown: 65533 closed tcp ports (reset)

PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 7.02 seconds
Raw packets sent: 69155 (3.043MB) | Rcvd: 65536 (2.621MB)
```

and  the site is snapped.htb and open the site looks like ,

![[Pasted image 20260604212350.png]]

and i try the subdomains , 

```
ffuf -u http://snapped.htb/ \
     -H "Host: FUZZ.snapped.htb" \
     -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
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
 :: URL              : http://snapped.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.snapped.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 302,404
________________________________________________

admin                   [Status: 200, Size: 1407, Words: 164, Lines: 50, Duration: 315ms]
```

the subdomain is admin.snapped.htb and this is Ngnix UI and try the posiible CVE  , and it is cd CVE-2026-27944 

`https://github.com/Skynoxk/CVE-2026-27944`  

Actually the CVE is about the Nginx UI is a web user interface for the Nginx web server. Prior to version 2.3.3, the /api/backup endpoint is accessible without authentication and discloses the encryption keys required to decrypt the backup in the X-Backup-Security response header. This allows an unauthenticated attacker to download a full system backup containing sensitive data (user credentials, session tokens, SSL private keys, Nginx configurations) and decrypt it immediately. This issue has been patched in version 2.3.3.

the Specific directory called `backup_extracted ` is extracted and this is about all the details about the back-up files  , 

the hash of the jonathon is 

`jonathan$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq` 

if we crack that , the passowrd is linkinpark

$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq:linkinpark

after that we got the jonathon shell  , and read the user flag , 

```
jonathan@snapped:~$ cat user.txt
29a6f3e734ab576e03562a12101a9e30

```


access  to the root,

