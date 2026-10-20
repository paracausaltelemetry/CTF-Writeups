---
title: Committed
summary: TryHackMe Git forensics room focused on recovering a sensitive flag from repository history after it was removed in a later commit.
date: 2026-06-29
tags: [TryHackMe, Git, Forensics, Source Control, Secrets]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/committed
---

# CTF Room: Committed
- [Link to room](https://tryhackme.com/room/committed)
- **Difficulty:** Easy
- **Category:** Git, Forensics, Source Control
- **OS:** Linux

## 1. Brief
Committed is a short TryHackMe room about recovering sensitive data from Git history.

The scenario is that a developer accidentally committed sensitive code to a repository and later removed it. Since Git keeps historical commits, the task is to inspect the repository history and recover the flag.

## 2. Lab Setup
The room provides an in-browser VM. The files are located under:

```bash
/home/ubuntu/commited
```

After opening the terminal, I moved into the extracted repository.

```bash
cd /home/ubuntu/commited/commited
```

## 3. Reviewing Git History
The first step was to inspect the commit history.

### Git Log
```bash
git log
```

### Output
```bash
commit ||28c36211be8187d4be04530e340206b856198a84|| (HEAD -> master)
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:49:32 2022 -0800

    Finished

commit ||9ecdc566de145f5c13da74673fa3432773692502||
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:40:19 2022 -0800

    Database management features added.

commit ||26bcf1aa99094bf2fb4c9685b528a55838698fbe||
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:32:49 2022 -0800

    Create database logic added

commit ||b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67||
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:30:43 2022 -0800

    Connecting to db logic added

commit ||441daaaa600aef8021f273c8c66404d5283ed83e||
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:28:16 2022 -0800

    Initial Project.
```

The normal log only showed the current branch, so I also checked all branches with a graph view.

### All Branches
```bash
git log --oneline --graph --all
```

### Output
```bash
* 28c3621 (HEAD -> master) Finished
| * 4e16af9 (dbint) Reminder Added.
| * c56c470 Oops
| * 3a8cc16 DB check
| * 6e1ea88 Note added
|/
* 9ecdc56 Database management features added.
* 26bcf1a Create database logic added
* b0eda7d Connecting to db logic added
* 441daaa Initial Project.
```

The `dbint` branch contained an interesting commit named `Oops`, which looked like the likely place where the sensitive value had been removed.

## 4. Recovering The Deleted Secret
I inspected the `Oops` commit directly.

### Git Show
```bash
git show c56c470
```

### Output
```bash
commit ||c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5||
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:46:39 2022 -0800

    Oops

diff --git a/main.py b/main.py
index 54d0271..0e1d395 100644
--- a/main.py
+++ b/main.py
@@ -4,7 +4,7 @@ def create_db():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root", # Username Goes Here
-    password="||flag{a489a9dbf8eb9d*******************||" # Password Goes Here
+    password="" # Password Goes Here
     )

     mycursor = mydb.cursor()
@@ -16,7 +16,7 @@ def create_tables():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root", #username Goes here
-    password="||flag{a489a9dbf8eb9d*******************||", #password Goes here
+    password="", #password Goes here
     database="commited"
     )

@@ -29,7 +29,7 @@ def populate_tables():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root",
-    password="||flag{a489a9dbf8eb9d*******************||",
+    password="",
     database="commited"
     )
```

The diff shows that the password field previously contained the flag and was later replaced with an empty string.

## 5. Flag

#### Discover the flag in the repository!
### Answer
```bash
||flag{a489a9dbf8eb9d*******************||
```

## 6. Summary
This room is a straightforward reminder that deleting a secret from the current version of a file does not remove it from Git history.

Using `git log --oneline --graph --all` revealed commits on another branch, and `git show` exposed the exact diff where the sensitive value was removed. The flag was still recoverable from that historical commit.
