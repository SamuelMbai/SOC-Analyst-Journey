# Introduction to Information Security & Penetration Testing Foundations

## 1. Introduction


## Assignment 2: Getting Started

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

On a Linux system, the shell is a program that takes input from the user via the keyboard and passes these commands to the operating system to perform a specific function. A shell may be obtained by exploiting a web application or network/service vulnerability, or by obtaining credentials and logging into the target host remotely.

There are three main types of shell connections:

| Shell Type        | Description                                                                                                                                                                                                      |
| :---------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Reverse Shell** | Initiates a connection back to a "listener" on our attack box.                                                                                                                                                   |
| **Bind Shell**    | "Binds" to a specific port on the target host and waits for a connection from our attack box.                                                                                                                    |
| **Web Shell**     | Runs operating system commands via the web browser (typically non-interactive or semi-interactive). It can be used to run single commands (e.g., leveraging a file upload vulnerability to upload a PHP script). |

### What is a Port?

Ports are virtual points where network connections begin and end. They are software-based and managed by the host operating system. Ports are associated with a specific process or service, allowing computers to differentiate between different traffic types (e.g., SSH traffic flows to a different port than HTTP web requests over the same connection).

#### Common Ports & Protocols Reference Table

| Port(s)       | Protocol        |
| :------------ | :-------------- |
| 20/21 (TCP)   | FTP             |
| 22 (TCP)      | SSH             |
| 23 (TCP)      | Telnet          |
| 25 (TCP)      | SMTP            |
| 80 (TCP)      | HTTP            |
| 161 (TCP/UDP) | SNMP            |
| 389 (TCP/UDP) | LDAP            |
| 443 (TCP)     | SSL/TLS (HTTPS) |
| 445 (TCP)     | SMB             |
| 3389 (TCP)    | RDP             |

### What is a Web Server?

A web server is an application running on the back-end server that handles all HTTP traffic from the client-side browser, routes it to the requested destination pages, and responds to the browser. Web servers typically run on ports **80** and **443**.

### OWASP Top 10

| Number | Category                                   | Description                                                                                                                                                                                           |
| :----- | :----------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1**  | Broken Access Control                      | Restrictions are not appropriately implemented to prevent users from accessing other users' accounts, viewing sensitive data, accessing unauthorized functionality, modifying data, etc.              |
| **2**  | Cryptographic Failures                     | Failures related to cryptography, which often lead to sensitive data exposure or system compromise.                                                                                                   |
| **3**  | Injection                                  | User-supplied data is not validated, filtered, or sanitized by the application (e.g., SQL injection, command injection, LDAP injection).                                                              |
| **4**  | Insecure Design                            | Security weaknesses that occur when the application is not designed with security in mind from the beginning.                                                                                         |
| **5**  | Security Misconfiguration                  | Missing appropriate security hardening across any part of the application stack, insecure default configurations, open cloud storage, or verbose error messages.                                      |
| **6**  | Vulnerable and Outdated Components         | Using components (both client-side and server-side) that are vulnerable, unsupported, or out of date.                                                                                                 |
| **7**  | Identification and Authentication Failures | Authentication-related attacks that target user identity, authentication mechanisms, and session management.                                                                                          |
| **8**  | Software and Data Integrity Failures       | Code and infrastructure that does not protect against integrity violations (e.g., relying upon plugins, libraries, or modules from untrusted repositories/CDNs).                                      |
| **9**  | Security Logging and Monitoring Failures   | Failures that prevent the detection, escalation, and timely response to active breaches. Without logging, breaches go unnoticed.                                                                      |
| **10** | Server-Side Request Forgery (SSRF)         | SSRF flaws occur whenever a web application fetches a remote resource without validating the user-supplied URL, allowing an attacker to coerce requests to unexpected internal/external destinations. |

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
```

Information Security (Infosec) is the practice of protecting data from unauthorized access, modification, misuse, disruption, or destruction. Its primary goal is to ensure the confidentiality, integrity, and availability of information systems and data.

---

## 2. Risk Management Process

I learned that risk management is a structured process that helps organizations protect data while ensuring that productivity is not disrupted.

The process involves five key steps:

- **Risk Identification** – Identifying potential threats and vulnerabilities.
- **Risk Analysis** – Evaluating the likelihood and impact of each risk.
- **Risk Evaluation & Prioritization** – Ranking risks based on severity and business impact.
- **Risk Treatment/Response** – Choosing how to handle risks:
  - Accept
  - Avoid
  - Mitigate (Control)
  - Transfer
- **Monitoring & Review** – Continuously tracking risks for changes over time.

This structured approach ensures that risks are managed proactively and efficiently.

---

## 3. Red Team vs Blue Team

In cybersecurity operations:

- The **Red Team** simulates attackers by attempting to exploit systems and find vulnerabilities.
- The **Blue Team** acts as defenders, focusing on detection, response, and system protection.

Both teams work together to improve an organization’s overall security posture.

---

## 4. Role of Penetration Testers

I learned that penetration testers (security assessors, network testers, web testers, or red teamers) are responsible for identifying security weaknesses such as:

- Vulnerabilities in systems and applications  
- Misconfigurations  
- Exposure of sensitive data  
- Weak authentication mechanisms  

A good penetration tester collaborates with clients to reproduce findings and provide clear remediation or mitigation guidance.

I also learned that testing approaches vary depending on organizational goals, including:

- White-box testing  
- Black-box testing  
- Phishing simulations  
- Targeted red-team exercises  

Accurate risk rating requires understanding the organization’s risk context and applying structured risk assessment principles.

These hands-on concepts form a foundation that applies directly to real-world cybersecurity environments.

---

## 5. Setting Up a Pentest Environment

### Hypervisor

A hypervisor is software that allows users to create and run virtual machines (VMs). It enables multiple operating systems to run on a single physical machine in isolated environments.

For this setup, we use a modified version of **Parrot Security OS (Pwnbox)** to build a local virtual machine. It can be installed using:

- ISO (Optical Disc Image)
- OVA (Open Virtual Appliance)

---

## 6. Connecting Using VPN

A Virtual Private Network (VPN) allows secure connection to a private network, enabling access to internal systems as if directly connected.

VPN works by routing the user’s internet traffic through a secure remote server instead of the local Internet Service Provider (ISP), improving privacy and secure access to restricted environments.

---

## Conclusion

This foundational module introduced key cybersecurity concepts including risk management, team roles in security operations, penetration testing fundamentals, and secure lab setup using virtualization and VPNs. These concepts form the basis for practical cybersecurity work in real-world environments.

