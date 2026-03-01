
## Level 11 → 12: ROT13

The file `data.txt` contains a string of text, but it's been put through ROT13 — a simple substitution cipher, where every letter is shifted 13 places forward in the alphabet. So `a` becomes `n`, `b` becomes `o`, and so on. Since the alphabet has 26 letters, applying ROT13 twice gets you back to the original.

It's not encryption by any real definition, but it's worth knowing because it still shows up in some places — usually to obscure spoilers or mildly sensitive text rather than actually protect anything.

The `tr` command handles this cleanly by mapping one set of characters to another:

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

The `tr` command takes every letter and swaps it with the one 13 positions away — uppercase and lowercase handled separately. Once I understood what ROT13 actually was, the command made more sense.

---

## Level 12 → 13: Layers of Compression

This one took a while. `data.txt` is a file that's been compressed multiple times, in different formats, one on top of another. The goal is to keep decompressing until you get to the actual password.

The first step is to work in a temporary directory since we'll be creating a bunch of files:

```bash
mkdir /tmp/newuser4400
cp data.txt /tmp/newuser4400/
cd /tmp/newuser4400
```

The file is also a hex dump, so it needs to be converted back to binary first using `xxd`:

```bash
xxd -r data.txt data.bin
```

From there it's a process of checking the file type, renaming it with the right extension, and decompressing — repeated several times:

```bash
file data.bin
# data.bin: gzip compressed data

mv data.bin data.gz
gzip -d data.gz

file data
# data: bzip2 compressed data

mv data data.bz2
bzip2 -d data.bz2

file data
# data: gzip compressed data

mv data data.gz
gzip -d data.gz

# this continues a few more times
mv data5.bin data.tar
tar -xf data.tar
# until...

mv data8.bin data.gz
gzip -d data.gz

file data
#ASCII text
cat data8
```

Honestly this level was more tedious than difficult, but it taught me to always check what a file actually is before assuming. The `file` command from level 4 came in handy here again. Also learned that `.tar` files are archives that bundle multiple files together, while `gzip` and `bzip2` are the actual compression formats — they're often used in combination.

---

## Level 13 → 14: Logging in With a Key File Instead of a Password

No password to find this time. Instead, there's a file called `sshkey.private` sitting right in the home directory. The goal was simple: use that private key to SSH into `bandit14`.

I'd read about public-key authentication before, but this was the first time I actually had to *use* it. The theory is elegant — the server already has your public key, you prove you have the matching private key, no password typing needed. 

I tried with the most commonly used command

```bash
ssh -i sshkey.private bandit14@localhost -p 2220
```

…immediately failed with the message:

```
!!! You are trying to log into this SSH server with a password on port 2220 from localhost.
!!! Connecting from localhost is blocked to conserve resources.
!!! Please log out and log in again.
```
I was staring at that and trying to figure out what went wrong. Turns out the Bandit server is blocking SSH connections from localhost. After a bit of analyzing, I figured out that the SSH key must be used from my own machine, not from inside the Bandit server

From my local terminal (after `exit`ing bandit13):

```bash
# Copied the key to my local machine
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .

#Changed the permissions
chmod 600 sshkey.private

# Logged in through my local machine
ssh -i sshkey.private -p 2220 bandit14@bandit.labs.overthewire.org
```

I hit Enter after the command and the banner appeared and I was suddenly at `bandit14@bandit:~$` without typing a single password!
Once inside:

```bash
cat /etc/bandit_pass/bandit14
```
This level was less about finding something hidden and more about understanding how SSH authentication actually works. I learned how to use the -i flag to authenticate with a private key instead of a password, why SSH keys need strict permissions (chmod 600), and how specifying the correct port with -p 2220 matters. The biggest takeaway was realizing that context matters — running the right command from the wrong place (localhost vs my own machine) can completely change the outcome.

---

## Level 14 → 15: Talking to a Port

The password for the next level is retrieved by sending the current password to port 30000 on localhost. This means connecting to a service running on the same machine and giving it some input.

`nc` (netcat) is the tool for this — it opens a raw connection to a host and port, letting you send and receive data directly:

```bash
nc localhost 30000
<password(from previous level)>
```

Netcat is one of those tools that comes up a lot in security contexts. At its core it just opens connections and moves data around, but that simplicity makes it useful for a lot of things — testing if a port is open, sending data to a service, or even setting up basic listeners. This was my first time actually using it, and the concept of talking directly to a port rather than through a browser or application felt like a useful thing to get comfortable with.

---

## Level 15 → 16: The Same Thing, but Encrypted

Same idea as the previous level — send the current password to a port (30001 this time) and get the next one back. The difference is that this port uses SSL, meaning the connection is encrypted. Regular `nc` won't work here because it doesn't handle SSL.

`openssl s_client` is the alternative — it connects the same way but handles the encryption layer:

```bash
openssl s_client -connect localhost:30001
```
After the connection is established, paste in the correct password from the previous level.

There's a lot of output when `openssl s_client` connects — certificate details, handshake information, and so on. Most of it can be ignored for this purpose; once it settles you just type or paste your input. The important takeaway is understanding that SSL/TLS is a layer that sits on top of a regular connection to encrypt the data going back and forth — something that's fundamental to how secure connections work on the web.

---
