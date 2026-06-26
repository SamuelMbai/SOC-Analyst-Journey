# Cyber Shujaa

## Module 01: Getting Started

---

## 1. Introduction

Information Security (Infosec) is the practice of protecting data from unauthorized access, changes, unlawful use, disruption, etc.

### Risk Management Process

Risk management is a structured process that helps organizations protect data while ensuring productivity is not disrupted. It involves five key steps:

1. Identifying potential risks
2. Analyzing their likelihood and impact
3. Evaluating and prioritizing them
4. Deciding on a response (*accept, avoid, control, or transfer*)
5. Monitoring risks continuously for changes

This systematic approach ensures risks are managed proactively and effectively.

### Red Team vs. Blue Team

* **Red Team:** Plays the attackers' role.
* **Blue Team:** Plays the defenders' part.

### Role of Penetration Testers

Security assessors, network, web, or red teamers help organizations find risks like vulnerabilities, misconfigurations, and sensitive data exposure across internal and external systems. A good tester works with clients to reproduce issues and provide clear mitigation or remediation guidance.

Assessments range from white-box tests to phishing campaigns and targeted red-team exercises, chosen to match the organization's goals. Accurately rating findings requires understanding the organization's risk context and the risk management process. The hands-on skills learned in this module apply directly across real-world environments.

### Setting Up a Pentest Distro

A hypervisor is software that allows us to create and run virtual machines (VMs). We use a modified version of Parrot Security (**Pwnbox**) to build a local virtual machine.

We can choose two installation formats:

* **ISO** (Optical disc image)
* **OVA** (Open Virtual Appliance)

---

## 2. Penetration Testing Basics

### What is a Shell?

...

---

## 3. Basic Tools

Essential tools used daily by information security professionals include SSH, Netcat, Tmux, and Vim.

### Using Network Connections & VPNs

A Virtual Private Network (VPN) allows us to connect to a private (internal) network and access hosts and resources as if we were directly connected to it. VPNs route your traffic through a target private server instead of your standard Internet Service Provider (ISP).

* **SSL VPN:** Uses a web browser as the VPN client.
* **Client-based VPN:** Requires explicit client software to establish the connection.

#### Connecting to HTB VPN Command

```bash
sammbai3@htb[/htb]$ sudo openvpn user.ovpn

Thu Dec 10 18:42:41 2020 OpenVPN 2.4.9 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL]
Thu Dec 18 18:42:41 2020 library versions: OpenSSL 1.1.1g 21 Apr 2020, LZO 2.10
Thu Dec 10 18:42:41 2020 Outgoing Control Channel Authentication: Using 256 bit message hash 'SHA256'
Thu Dec 10 18:42:41 2020 Incoming Control Channel Authentication: Using 256 bit message hash 'SHA256'
Thu Dec 10 18:42:41 2020 TCP/UDP: Preserving recently used remote address: [AF_INET]
Thu Dec 10 18:42:41 2020 Socket Buffers: R=[212992->212992] S=[212992->212992]
Thu Dec 10 18:42:41 2020 UDP Link local: (not bound)
<SNIP>
Thu Dec 10 18:42:41 2020 Initialization Sequence Completed
```

### Using SSH

Secure Shell (SSH) runs on port 22 by default and provides a secure way to access a computer remotely.

```bash
sammbai3@htb[/htb]$ ssh Bob@10.10.10.10
Bob@remotehost's password: *********
Bob@remotehost#
```

### Using Netcat

Netcat (nc or ncat) is an excellent network utility for interacting with TCP/UDP ports, widely used to catch or connect to shells.

```bash
sammbai3@htb[/htb]$ netcat 10.10.10.10 22
SSH-2.0-OpenSSH_8.4p1 Debian-3
```

### Using Tmux

Terminal multiplexers allow expanding a standard Linux terminal's features, such as split layouts or maintaining multiple active windows within a single terminal session.

### Using Vim

Vim is a keyboard-driven terminal text editor. Relying entirely on the keyboard significantly increases your productivity once mastered. Press : to enter command mode at the bottom of the window.

---

## 4. Service Scanning & Target Enumeration

### Nmap Scanning

We use Nmap to discover open ports and running services on a target host. The -sC parameter specifies that default Nmap scripts should be executed to obtain deep information, while -sV determines version info.

```bash
sammbai3@htb[/htb]$ nmap 10.129.42.253

PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

### Detailed Advanced Script Scan

```bash
sammbai3@htb[/htb]$ nmap -sV -sC -p- 10.129.42.253

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.1
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: PHP 7.4.3 phpinfo()
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

### Banner Grabbing (Manual)

We can perform banner grabbing manually using Netcat to verify service headers directly:

```bash
sammbai3@htb[/htb]$ nc -nv 10.129.42.253 21
(UNKNOWN) [10.129.42.253] 21 (ftp) open
220 (vsFTPd 3.0.3)
```

### Server Message Block (SMB)

SMB is highly prevalent on networks and provides a useful environment for enum. We can run specific targeted scripts:

```bash
sammbai3@htb[/htb]$ nmap --script smb-os-discovery.nse -p445 10.10.10.40

Host script results:
| smb-os-discovery:
|   OS: Windows 7 Professional 7601 Service Pack 1
|   Computer name: CED-PC
|_  Workgroup: WORKGROUP
```

#### Listing Shares with smbclient

```bash
sammbai3@htb[/htb]$ smbclient -N -L \\\\10.129.42.253

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
users           Disk
IPC$            IPC       IPC Service (gs-svcscan server)
```

This scan reveals a non-default network share called users.

### SNMP Enumeration

SNMP Community strings provide configuration statistics. In SNMP v1 and v2c, strings are sent in plaintext. If known, information can be polled with snmpwalk:

```bash
sammbai3@htb[/htb]$ snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0

iso.3.6.1.2.1.1.5.0 = STRING: "gs-svcscan"
```

### Section Quiz Answers

**Q:** What does Nmap display as the version of the service running on port 8080?

**Answer:** Apache Tomcat

**Q:** Identify the non-default port that the telnet service is running on.

**Answer:** 2323

**Q:** Connect to the available share as the bob user, access the 'flag' folder, and submit flag.txt contents.

**Answer:** dceece590f3284c3866305eb2473d099

---

## 5. Web Enumeration

### GoBuster

GoBuster is used to perform directory brute-forcing/enumeration to find hidden files, administration backends, or exposed sensitive routes.

```bash
gobuster dir -u http://10.129.42.253 -w /usr/share/wordlists/dirb/common.txt
```

### WhatWeb

Used to easily finger-print the version of web servers, backing content management systems, plugins, and active development frameworks from the terminal layout.

### Robots.txt

Instructs search engines which pages to ignore. Reading it manually frequently leaks sensitive directories or developer testing scripts directly to security assessors.

### Section Quiz Answer

**Q:** Run web enumeration techniques on the target server to get the flag.

**Answer:** HTB{w3b_3num3r4710n_r3v34l5_53cr375}

---

## 6. Public Exploits & Metasploit Framework

...

---

## 7. Privilege Escalation & File Transfer

...

---

## 8. Hands-On Target: Nibbles Box Walkthrough

...

---

## 9. Final Module Knowledge Check

...

---

## 10. Conclusion

This module provided a strong foundation and boosted my confidence in ethical hacking by guiding me through gaining footholds, capturing flags, and documenting results effectively. Along the way, I developed a clear understanding of key concepts in Information Security, including the CIA triad (Confidentiality, Integrity, and Availability) and the distinct operational profiles defining red and blue team units.

Overall, the hands-on approach combined with theoretical knowledge has equipped me with the essential skills and mindset to continue progressing in penetration testing and cybersecurity.
