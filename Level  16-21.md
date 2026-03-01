
## Level 16 → 17: Finding the Right Port

The task here is to find which port on localhost, in the range 31000–32000, is actually running an SSL service — and then send the current password to it to get the next credentials.

Rather than trying every port manually, `nmap` can scan the range and tell us which ones are open and what they're running:

```bash
nmap -sV localhost -p 31000-32000
```

```
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown
31960/tcp open  echo
```

A few ports came back. I ignored ssl/echo because an echo service simply sends back whatever input it receives. Port 31790, running SSL without just echoing input back, looked interesting. Connected to it using `ncat` with the `--ssl` flag:

```bash
ncat --ssl localhost 31790
```
Typed in the current password and got back a private SSH key. Copied it, exited the session, and saved the key to a file on my local machine:

```bash
mkdir /tmp/level16
vim /tmp/level16/private.key
# pasted the key in and saved
chmod 600 /tmp/level16/private.key
```
The `chmod 600` is important — SSH will flat out refuse to use a key file if the permissions are too open. Learned that one the slightly annoying way.

Then tried to SSH in using the key:

```bash
ssh -i /tmp/level16/private.key bandit17@localhost -p 2220
```

```
ssh: connect to host localhost port 2220: Connection refused
```

That didn't work because I was running this from my own machine, not from inside the bandit server — so `localhost` was pointing to my machine, not the game server. Fixed it by using the actual server address:

```bash
ssh -i /tmp/level16/private.key bandit17@bandit.labs.overthewire.org -p 2220
```

That connected successfully and dropped me into bandit17's shell.

The main thing I took away here is the difference between `ncat` and `openssl s_client` — both can handle SSL connections, `ncat --ssl` is just a bit cleaner to use. 

---
## Level 17 → 18: Spotting the Difference Between Two Files

There are two files in the home directory — `passwords.old` and `passwords.new`. The password for the next level is the one line in `passwords.new` that doesn't appear in `passwords.old`.

`diff` compares two files line by line and shows what changed:

```bash
diff passwords.old passwords.new
```

The `<` and `>` symbols indicate which file each line belongs to — `<` is the old file, `>` is the new one. 

Short level, but `diff` is a genuinely useful command to know. It's used constantly to comparing config files, checking what changed between two versions of something, reviewing patches. Worth getting familiar with.

---

## Level 18 → 19: Bypassing a Modified Shell Config

Logging in as bandit18 immediately drops the connection. The `.bashrc` file — which runs automatically when a bash session starts — has been modified to log you out as soon as you connect.

The way around this is to run a command directly over SSH without letting bash start an interactive session:

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

When I passed a command as an argument to SSH it ran it on the remote machine and return the output, without ever starting a proper shell session. Since `.bashrc` never runs, the logout trap never triggers. This was a good demonstration of how startup scripts can be abused.
---

## Level 19 → 20: Using a Setuid Binary

There's a file in the home directory called `bandit20-do`.
```bash
bandit19@bandit:~$ ls -la
total 36
drwxr-xr-x   2 root     root      4096 Oct 14 09:26 .
drwxr-xr-x 150 root     root      4096 Oct 14 09:29 ..
-rwsr-x---   1 bandit20 bandit19 14884 Oct 14 09:26 bandit20-do
-rw-r--r--   1 root     root       220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root     root      3851 Oct 14 09:19 .bashrc
-rw-r--r--   1 root     root       807 Mar 31  2024 .profile
```
Running it with no arguments gives a usage hint:

```bash
./bandit20-do
```

```
Run a command as another user.
  Example: ./bandit20-do whoami
```
It's a setuid binary — a program that runs with the permissions of its owner rather than the person executing it. In this case it's owned by bandit20, so anything run through it executes as bandit20. That means we can use it to read a file we'd normally not have access to:

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

Setuid binaries are worth understanding because they're a common area of interest in privilege escalation. When a program is setuid and has a vulnerability, it can sometimes be exploited to run arbitrary commands with elevated permissions. Here it's intentional and controlled, but the concept translates directly to real-world scenarios.

---

## Level 20 → 21: A Setuid Binary That Talks to a Port

This one builds on the previous level. There's another setuid binary called `suconnect`. It connects to a port on localhost, and if it receives the current password for bandit20, it sends back the password for bandit21.

```bash
$ ls -la
total 36
drwxr-xr-x   2 root     root      4096 Oct 14 09:26 .
drwxr-xr-x 150 root     root      4096 Oct 14 09:29 ..
-rw-r--r--   1 root     root       220 Mar 31  2024 .bash_logout
-rw-r--r--   1 root     root      3851 Oct 14 09:19 .bashrc
-rw-r--r--   1 root     root       807 Mar 31  2024 .profile
-rwsr-x---   1 bandit21 bandit20 15608 Oct 14 09:26 suconnect

```
The catch was that I need to set up something listening on that port first. This is where `nc` in listen mode comes in. The trick is to run both things at the same time — a listener in the background and the binary in the foreground:

```bash
echo "<password>" | nc -lp 1234 &
./suconnect 1234
```

```
Read: <previous password>
Password matches, sending next password
<new password>
```

The `&` at the end of the first command runs it in the background, freeing up the terminal to run `suconnect`. The listener sends the password when `suconnect` connects, `suconnect` verifies it and sends back the next one. Setting up a listener with netcat was something I'd read about but hadn't actually done before — it's a pretty fundamental networking concept and it's good to have seen it work firsthand.

---
