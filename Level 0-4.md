# OverTheWire: Bandit — Writeups (Levels 0–4)

*My notes from working through the Bandit wargame. These writeups are mostly me documenting what I tried, what broke, and what I learned from it.*

---

## Level 0 → 1: Getting Connected

The first level is just about logging in over SSH and finding a file. I'd used SSH once or twice before but never with a custom port, so the `-p 2220` part was new to me. Turns out the server runs on port 2220 instead of the usual 22 — apparently this is something people do to reduce automated noise that knock on the default port.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

Once I was in, I listed the files in the home directory and found one called `readme`. Read it, got the password.

```bash
ls -la
cat readme
```

What stuck with me: a password just sitting in a plain text file feels wrong, and it is. In a real system, that would be a problem — anyone who gets access to that directory gets the password too. Good reminder that hiding something is not the same as securing it.

---

## Level 1 → 2: A File Named `-`

This one caught me off guard. The password is in a file literally called `-`, and when I ran `cat -` it just... hung. Nothing happened. I had to Ctrl+C out of it.

After a bit of googling I found out that `-` has a special meaning in the terminal — a lot of commands treat it as "read from keyboard input" rather than a file. So the shell was waiting for me to type something, not reading the file.

The fix is to give it a proper path so it knows you mean the file:

```bash
cat ./-
```

I wouldn't have thought a filename could cause that kind of confusion. It made me realise that the terminal interprets what you type in ways that aren't always obvious, and little things like a dash or a dot actually mean something.

---

## Level 2 → 3: Spaces in the Filename

The file here is called `--spaces in this filename--`. I typed `cat --spaces in this filename--` and got a bunch of "no such file or directory" errors — one for each word. The terminal treated each space as a separator and thought I was asking for several different files.

Wrapping the name in quotes solved it:

```bash
cat "./--spaces in this filename--"
```


Simple fix once you know it, but it's one of those things that would have had me staring at the screen for a while without a nudge in the right direction. I can see how spaces in filenames would break a lot of scripts if you're not careful about quoting.

---

## Level 3 → 4: The File That Wasn't There

Navigated into a folder called `inhere`, ran `ls`, and got nothing. Empty directory. But the challenge said there was a file in there, so I tried `ls -la` instead — which shows hidden files — and there it was.

```bash
ls -la
# ...Hiding-From-You
```

```bash
cat ...Hiding-From-You
```


In Linux, any file that starts with a dot is hidden from the regular `ls` view. I knew about dotfiles in a vague sense (like `.bashrc`) but hadn't thought about it as something that could be used to hide things. Makes sense that `ls -la` should probably just be the default habit — you don't want to miss things.

---

## Level 4 → 5: Finding the Right File

There were ten files in the folder — `-file00` to `-file09` — and I needed to find the one with readable text. I could have just opened each one, but the level description mentioned something about human-readable file. After a bit of googling I found out a way to find the filetype using `file` command, which tells you what kind of content a file actually contains. Gave it a shot:

```bash
file ./*
```

```
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```

`-file07` was the only one with plain text, so:

```bash
cat ./-file07
```

The `file` command was new to me. It doesn't go by the filename or extension — it looks at the actual contents of the file to figure out what it is. That's a useful thing to know. Also just a good reminder to look for the right tool before doing something the slow way.

---

*More levels to come as I work through them.*
