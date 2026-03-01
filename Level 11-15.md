
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

No password to find this time. Instead, there's a private SSH key file in the home directory called `sshkey.private`. The goal is to use it to log into the next level's account.

SSH supports key-based authentication as an alternative to passwords — the server checks whether your key matches the one it expects, rather than asking you to type a password. It's actually considered more secure than password login when set up properly.

```bash
ssh -i sshkey.private bandit14@localhost -p 2220
```

Once in as bandit14, the password for this level is stored at a known path:

```bash
cat /etc/bandit_pass/bandit14
```

The `-i` flag tells SSH which key file to use. I hadn't done key-based SSH before this — it felt a bit different from just typing a password, but once it worked it made sense. Key files need to have the right permissions too, otherwise SSH refuses to use them (`chmod 600` on the key file is usually required).

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
After the connection is established, paste in the password from the previous level.

There's a lot of output when `openssl s_client` connects — certificate details, handshake information, and so on. Most of it can be ignored for this purpose; once it settles you just type or paste your input. The important takeaway is understanding that SSL/TLS is a layer that sits on top of a regular connection to encrypt the data going back and forth — something that's fundamental to how secure connections work on the web.

---
