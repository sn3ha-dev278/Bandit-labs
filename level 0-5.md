# Level 0 - 1

# 1. Overview
**Level Goal (Bandit 0 -> 1):**  
-> Log into the game using SSH.

# 2. Reconnaissance / Information Gathering

This stage involves gathering the information about the target.

- **Host:** bandit.labs.overthewire.org  
- **Port:** 2220 (non-standard SSH port)  
- **Protocol:** SSHv2  
- **Authentication:** Password  
- **Initial Credentials:** bandit0:bandit0 (publicly provided - a “known credential” scenario)

# 3. Initial Access

### Step 1: Establish SSH Session

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

- Enter the password: bandit0  

Success message:

```bash
This is an OverTheWire game server.
More information on http://www.overthewire.org/wargames
...
```

---

### Step 2: Post-access Recon

```bash
whoami          
pwd            
ls -la         
```

Output:

```bash
bandit0@bandit:~$ ls -la
total 24
drwxr-xr-x   2 root    root    4096 Oct 14 09:26 .
drwxr-xr-x 150 root    root    4096 Oct 14 09:29 ..
-rw-r--r--   1 root    root     220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root    root    3851 Oct 14 09:19 .bashrc
-rw-r--r--   1 root    root     807 Mar 31  2024 .profile
-rw-r-----   1 bandit1 bandit0  438 Oct 14 09:26 readme
```

---

### Step 3: Extract the Credential

```bash
cat readme
```

**Password for Bandit 1:**  
ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If

---

### Step 4: Clean Exit

```bash
exit
```

# 4. Command Analysis

- whoami : verifies the current effective user  
- pwd : confirm the expected home directory  
- ls -la : hidden files often contain sensitive information  
- cat : read file content  
- exit : explicitly logout  

# 5. Security Takeaways

1. SSH Hygiene – Moving SSH to non-standard ports reduces automated scanning noise but does not provide real security.
2. Least Privilege – Each Bandit user starts with zero privileges except read access to their own home directory  
3. Credential Management – Passwords stored in plaintext files are a classic misconfiguration  
4. Minimal tools – Only "ssh", "ls", and "cat" were required.  

---

# Level 1 - 2

# 1. Overview
**Level Goal (Bandit 1 → 2):**  
→ The password for the next level is stored in a file called `-` located in the home directory.

# 2. Reconnaissance / Information Gathering

(Same as the previous level)

- **Initial Credentials:** bandit1:ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If (from previous level)

# 3. Initial Access

### Step 1: Establish SSH Session

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

- Enter the password: ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If  
- Success message: (same welcome banner)  
- User is now logged in as "bandit1"

---

### Step 2: Post-access Recon

```bash
whoami          
pwd            
ls -la         
```

Output example:

```bash
bandit1@bandit:~$ ls -la
total 24
drwxr-xr-x   2 root    root    4096 ...
drwxr-xr-x 150 root    root    4096 ...
-rw-r--r--   1 root    root     220 ...
-rw-r--r--   1 root    root    3851 ...
-rw-r--r--   1 root    root     807 ...
-rw-r-----   1 bandit2 bandit1  ...  -
```

---

### Step 3: Extract the Credential

##### WRONG WAY - hangs / waits for input:

```bash
cat -
```

##### CORRECT WAY:

```bash
cat ./-
```

**Password for Bandit 2:**  
263JGJPfgU6LtdEvgfWU1XP5yac29mFx

---

### Step 4: Clean Exit

```bash
exit
```

# 4. Command Analysis

- whoami : current user  
- pwd : working directory  
- ls -la : show all files including dotfiles and special names  
- cat ./- : read file whose name starts with dash (./ forces path interpretation)  
- exit : logout  

# 5. Security Takeaways

1. Special filenames – Files named -, --help, etc. break many commands as leading "-" is parsed as option  
2. Defensive filename handling – Using ./ or full path when dealing with user-controlled filenames  
3. Least Privilege – bandit1 can only read its own file 
4. Credential hygiene – plaintext passwords in home directories  
5. Argument parsing - Common source of vulnerabilities in command-line tools when user-controlled input is passed directly into system commands.  

---

# Level 2 - 3

# 1. Overview
**Level Goal (Bandit 2 → 3):**  
→ The password for the next level is stored in a file called "--spaces in this filename--" located in the home directory.

# 2. Reconnaissance / Information Gathering

(Same as the previous levels)

- **Initial Credentials:** bandit2:263JGJPfgU6LtdEvgfWU1XP5yac29mFx

# 3. Initial Access

### Step 1: Establish SSH Session

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220
```

- Enter the password: 263JGJPfgU6LtdEvgfWU1XP5yac29mFx  
- Success message is displayed  
- User is now logged in as "bandit2"

---

### Step 2: Post-access Recon

```bash
whoami          
pwd             
ls -la          
```

Output example:

```bash
bandit2@bandit:~$ ls -la
total 24
drwxr-xr-x   2 root    root    4096 Oct 14 09:26 .
drwxr-xr-x 150 root    root    4096 Oct 14 09:29 ..
-rw-r--r--   1 root    root     220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root    root    3851 Oct 14 09:19 .bashrc
-rw-r--r--   1 root    root     807 Mar 31  2024 .profile
-rw-r-----   1 bandit3 bandit2   33 Oct 14 09:26 --spaces in this filename--
```

---

### Step 3: Extract the Credential

#### WRONG WAY (fails as it treats each word as separate argument):

```bash
cat --spaces in this filename--
cat: spaces: No such file or directory
cat: in: No such file or directory
...
```

##### CORRECT WAYS:

```bash
cat ./--spaces\ in\ this\ filename--
```

**Password for Bandit 3:**  
MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

---

### Step 4: Clean Exit

```bash
exit
```

# 4. Command Analysis

- whoami : current user  
- pwd : working directory  
- ls -la : shows all files including those with long/special names  
- cat ./--spaces\ in\ this\ filename-- : ./ prevents the leading dashes from being treated as command options and \ is used to escape the spaces.  
- cat file\ with\ spaces : escapes each space with backslash  
- exit : logout  

# 5. Security Takeaways

1. Filenames with spaces – Dangerous when passed without quotes to commands as shell word-splitting breaks them into multiple arguments  
2. Shell escaping – Escape user-controlled filenames/paths to prevent injection-like misinterpretation  
3. Least Privilege – bandit2 can only read its own file with spaces 
4. Credential hygiene – plaintext passwords in home directories  
5. Argument parsing - One of the command-line vulnerabilities due to unexpected behavior  

---

# Level 3 - 4

# 1. Overview
**Level Goal (Bandit 3 → 4):**  
→ The password for the next level is stored in a hidden file in the inhere directory.

# 2. Reconnaissance / Information Gathering

(Same as the previous levels)

- **Initial Credentials:** bandit3:MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

# 3. Initial Access

### Step 1: Establish SSH Session

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

- Enter the password: MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx  
- Success message is displayed  
- User is now logged in as "bandit3"

---

### Step 2: Post-access Recon

```bash
whoami         
pwd             
ls -la          
cd inhere
ls -la          
```

Output example:

```bash
bandit3@bandit:~$ ls -la
total 24
drwxr-xr-x   3 root root 4096 Oct 14 09:26 .
drwxr-xr-x 150 root root 4096 Oct 14 09:29 ..
-rw-r--r--   1 root root  220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root root 3851 Oct 14 09:19 .bashrc
drwxr-xr-x   2 root root 4096 Oct 14 09:26 inhere
-rw-r--r--   1 root root  807 Mar 31  2024 .profile
```

```bash
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Oct 14 09:26 .
drwxr-xr-x 3 root    root    4096 Oct 14 09:26 ..
-rw-r----- 1 bandit4 bandit3   33 Oct 14 09:26 ...Hiding-From-You
```

---

### Step 3: Extract the Credential

#### WRONG WAY (hidden file not shown with normal ls):

```bash
ls
```
(nothing is displayed)

#### CORRECT WAY:

```bash
ls -a
cat ...Hiding-From-You
```

**Password for Bandit 4:**  
2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

---

### Step 4: Clean Exit

```bash
exit
```

# 4. Command Analysis

- whoami : current user  
- pwd : working directory  
- ls -la : shows all files including hidden ones  
- cd inhere : change the directory  
- cat ...Hiding-From-You : reads the hidden file content  
- exit : logout  

# 5. Security Takeaways

1. Hidden files – In Unix/Linux, files starting with . (dotfiles) are hidden from normal ls output  
2. Enumeration – Using ls -a  or ls -la reveals hidden files not shown by default  
3. Least Privilege – bandit3 can only read its own .hidden file  
4. Credential hygiene – plaintext passwords in (even hidden) home directory files expose sensitive information


---

# Level 4 - 5

# 1. Overview
**Level Goal (Bandit 4 → 5):**  
→ The password for the next level is stored in the only human-readable file in the inhere directory

# 2. Reconnaissance / Information Gathering

(Same as the previous levels)

- **Initial Credentials:** bandit4:2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

# 3. Initial Access

### Step 1: Establish SSH Session

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
```

- Enter the password: 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ  
- Success message is displayed  
- User is now logged in as "bandit4"

---

### Step 2: Post-access Recon

```bash
whoami         
pwd             
ls -la          
cd inhere
ls -la          
```

Output example:

```bash
bandit3@bandit:~$ ls -la
total 24
drwxr-xr-x   3 root root 4096 Oct 14 09:26 .
drwxr-xr-x 150 root root 4096 Oct 14 09:29 ..
-rw-r--r--   1 root root  220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root root 3851 Oct 14 09:19 .bashrc
drwxr-xr-x   2 root root 4096 Oct 14 09:26 inhere
-rw-r--r--   1 root root  807 Mar 31  2024 .profile
```

```bash
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Oct 14 09:26 .
drwxr-xr-x 3 root    root    4096 Oct 14 09:26 ..
-rw-r----- 1 bandit4 bandit3   33 Oct 14 09:26 ...Hiding-From-You
```

---

### Step 3: Extract the Credential

#### WRONG WAY (hidden file not shown with normal ls):

```bash
ls
```
(nothing is displayed)

#### CORRECT WAY:

```bash
ls -a
cat ...Hiding-From-You
```

**Password for Bandit 4:**  
2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

---

### Step 4: Clean Exit

```bash
exit
```

# 4. Command Analysis
- whoami : current user  
- pwd : working directory  
- ls -la : shows all files including hidden ones  
- cd inhere : change the directory  
- cat ...Hiding-From-You : reads the hidden file content  
- exit : logout  

# 5. Security Takeaways
1. Hidden files – In Unix/Linux, files starting with . (dotfiles) are hidden from normal ls output  
2. Enumeration – Using ls -a  or ls -la reveals hidden files not shown by default  
3. Least Privilege – bandit3 can only read its own .hidden file  
4. Credential hygiene – plaintext passwords in (even hidden) home directory files expose sensitive information
