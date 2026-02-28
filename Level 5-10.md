
## Level 5 → 6: Finding a File by Properties

This level had a whole directory full of folders, each with multiple files inside. The password is in there somewhere, but the challenge gives you a few clues about what the file looks like — it's human-readable, 1033 bytes in size, and not executable.

Opening files one by one clearly wasn't the move. I looked up how to search by file properties and landed on the `find` command, which lets you filter by things like size and type:

```bash
find . -type f -size 1033c ! -executable
```

```
./maybehere07/.file2
```

```bash
cat ./maybehere07/.file2
```

The `find` command felt like a big unlock. Before this I was only using it in the most basic way, but being able to filter by size, permissions, and file type makes it a lot more useful. The `-size 1033c` part tripped me up at first — the `c` means bytes, which isn't obvious.

---

## Level 6 → 7: Searching the Whole Server

Similar idea to the last level, but this time the file could be anywhere on the server, not just in the home directory. The hints were that it's owned by a specific user (`bandit7`) and group (`bandit6`), and is 33 bytes in size.

Starting the search from the root `/` directory:

```bash
find / -type f -size 33c -user bandit7 -group bandit6
```

```
/var/lib/dpkg/info/bandit7.password
```

```bash
cat /var/lib/dpkg/info/bandit7.password
```

The terminal was flooded with "permission denied" errors from folders I don't have access to. The file was located in the directory 
/var/lib/dpkg/info/

---

## Level 7 → 8: Searching Inside a File

The password is in a file called `data.txt`, which has thousands of lines in it. The hint says the password is next to the word "millionth". There was no way I was going to scroll through that manually. This is where I used `grep` command to search through a file and returns lines matching a word or pattern:

```bash
grep "millionth" data.txt
```

That's it. One command, instant result. `grep` is one of those tools that sounds simple but comes up constantly. Any time you need to find something in a big file or a long output, it's the go-to.

---

## Level 8 → 9: The Line That Only Appears Once

`data.txt` again, but this time the password is the only line in the file that doesn't repeat. Every other line appears multiple times.
I looked this one up because I wasn't sure how to approach it. The solution uses two commands chained together — `sort` and `uniq`:

```bash
sort data.txt | uniq -u
```

`sort` rearranges the lines alphabetically, which groups all the duplicates together. Then `uniq -u` goes through and only keeps lines that appear exactly once. The `|` in the middle is a pipe — it takes the output of `sort` and feeds it straight into `uniq`. I'd seen pipes mentioned before but this was the first time I actually used one properly. Pretty useful once it clicks.

---

## Level 9 → 10: Strings Hidden in a Binary File

`data.txt` this time is not a normal text file — opening it with `cat` printed a bunch of garbled characters. The challenge says the password is one of the few human-readable strings in the file, and it's preceded by several `=` signs.

The `strings` command pulls out any readable text it can find inside a file, even if most of the file is unreadable:

```bash
strings data.txt | grep "==="
```

I didn't know the `strings` command existed before this. It's apparently used a lot when people are trying to analyse programs or files where you can't just read the contents normally — you use `strings` to fish out whatever readable bits are in there. Combining it with `grep` to filter for the `=` signs made it much cleaner than reading through everything it returned.

---

## Level 10 → 11: Base64 Encoding

`data.txt` has a single line of text that looks like random characters. The challenge says it's encoded in base64.

Base64 is a way of converting data into a string of plain letters and numbers, a different way of representing the same information. It is seen a lot when data is being passed around in places that only handle text. Decoding it was straightforward:

```bash
base64 -d data.txt
```

This was a good one to run into early because base64 shows up everywhere — in web requests, in emails, in configuration files. Knowing how to spot it and decode it feels like a genuinely useful thing to have in the back of my head.

---
