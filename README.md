# OverTheWire: Bandit — Writeups

A collection of writeups documenting my progress through the [Bandit wargame](https://overthewire.org/wargames/bandit/) by OverTheWire. Bandit focuses on building foundational Linux and command-line skills that are directly relevant to security work.

I started this as someone fairly new to ethical hacking. The writeups aren't meant to be perfect walkthroughs — they're honest notes on what I tried, what didn't work, and what I learned along the way.

---

## What is Bandit?

Bandit is a beginner-friendly wargame where each level presents a challenge that requires finding a password to unlock the next one. The challenges cover things like navigating the Linux filesystem, working with files and permissions, using network tools, and understanding how basic programs behave.

---

## Tools and Commands Covered

These are the tools and concepts that came up across the levels.

| Tool / Concept | What it's used for |
|---|---|
| `ssh` | Connecting to remote machines securely |
| `ls`, `cat`, `cd` | Basic filesystem navigation and file reading |
| `file` | Identifying file types by content, not extension |
| `find` | Searching for files by name, size, permissions, ownership |
| `grep` | Searching for patterns inside files or command output |
| `sort`, `uniq` | Sorting and deduplicating lines in a file |
| `strings` | Extracting readable text from binary files |
| `base64` | Encoding and decoding base64 data |
| `tr` | Translating or substituting characters (used for ROT13) |
| `xxd` | Hex dumping and reversing hex dumps |
| `gzip`, `bzip2`, `tar` | Compressing and decompressing files |
| `diff` | Comparing two files line by line |
| `nc` (netcat) | Opening raw network connections, setting up listeners |
| `nmap` | Scanning ports and detecting running services |
| `ncat` / `openssl s_client` | Connecting to SSL-encrypted services |
| `chmod` | Changing file permissions |
| Cron jobs | Scheduled tasks and how they can be misconfigured |
| Setuid binaries | Programs that run with elevated permissions |
| Shell quoting | Handling special characters and spaces in filenames |
| `more` / `vim` | Pager and editor abuse for shell escape |

---

## Notes
- Levels are tackled on Kali Linux.
- Some levels were solved by reading documentation (`man` pages) and a bit of googling — that's noted where relevant.
---
