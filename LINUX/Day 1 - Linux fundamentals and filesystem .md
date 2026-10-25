# Linux Day 1: Fundamentals and Filesystem

**Goal:** Understand what Linux is and move around the filesystem confidently.

---

## 1. What is Linux?

### The four words you must know

| Term | Simple meaning | Analogy (restaurant) |
|---|---|---|
| **Kernel** | The core of the system. Talks to the hardware (CPU, memory, disk) and decides which program gets what. | The kitchen |
| **Shell** | A program that reads the commands you type and asks the kernel to run them. Common shells: bash, zsh, sh. | The waiter taking your order |
| **Terminal** | The window where you type and see text. It runs a shell inside it. | Your table and menu |
| **User space** | Everything that is not the kernel: the shell, commands like `ls`, and apps like nginx. They ask the kernel for help through *system calls*. | The dining room |

**Key point:** strictly speaking, Linux is only the kernel. A **distribution (distro)** is the kernel plus tools, a package manager and defaults.

### Common distributions

| Family | Examples | Package tool | Where you will see it |
|---|---|---|---|
| Debian family | Ubuntu, Debian | `apt` | Most tutorials, laptops, cloud servers |
| Red Hat family | RHEL, Rocky, AlmaLinux, Fedora | `dnf` | Enterprise servers |
| Amazon Linux | Amazon Linux 2023 | `dnf` | Default on AWS EC2 |
| Alpine | Alpine | `apk` | Tiny Docker containers |

Use **Ubuntu** for learning. The basics are the same everywhere.

Check which distro you have:

```bash
cat /etc/os-release
```

---

## 2. The filesystem hierarchy

Linux has **one single tree** that starts at `/` (called "root"). There are no drive letters like `C:`. Disks are attached (mounted) somewhere inside the tree. Almost everything is treated as a file.

**Mounting:** each disk or partition is attached ("mounted") at a folder in the tree, called a *mount point*. Run `df -h` to see what is mounted where, or `lsblk` to list your disks. Extra disks and USB drives usually appear under `/mnt` or `/media`.

**Everything is a file:** documents, folders, devices and even live system information are read and written like files. Try `cat /proc/cpuinfo` to read live CPU details. `/dev/null` is a device you can write to like a file; it simply throws away whatever you send it.

| Directory | What it is for | Example |
|---|---|---|
| `/` | The top of the tree. Everything lives under it. (Not the same as `/root`.) | |
| `/home` | Personal files of normal users. Each user gets a folder. | `/home/pav` |
| `/etc` | System **configuration** files, mostly plain text. | `/etc/hostname`, `/etc/ssh/` |
| `/var` | Data that **changes** while the system runs: logs, caches, mail. | `/var/log` |
| `/tmp` | It is commonly writable by all users, with permissions controlling access to individual files. | |
| `/usr` | Installed programs, libraries and shared data. | `/usr/bin` |
| `/opt` | **Optional** third-party software that brings its own folder. | `/opt/google` |
| `/dev` | **Device** files that represent hardware. | `/dev/null`, `/dev/sda` |
| `/proc` | A **virtual** folder with live info about processes and the kernel. Not on disk. | `/proc/cpuinfo` |
| `/run` | Runtime data since boot (process IDs, sockets). Lives in memory. | |

Others worth knowing: `/root` (home of the root user), `/boot` (files to start the system), `/bin` and `/sbin` (on modern distros these point to `/usr/bin` and `/usr/sbin`).

### `/bin` vs `/sbin`

- `/bin` holds **essential commands for all users**, such as `ls`, `cp`, `cat` and `mkdir`.
- `/sbin` holds **system administration commands**, mostly meant for the root user, such as `reboot`, `fdisk` and `fsck`.
- `/usr/bin` and `/usr/sbin` follow the same split for the rest of the installed programs.
- On modern distros, `/bin` and `/sbin` are just shortcuts (symbolic links) to `/usr/bin` and `/usr/sbin`, so both paths work. Check with `ls -ld /bin /sbin`; the arrow `->` in the output shows the link.

*Memory trick:* the **s** in `sbin` stands for **system** (admin) commands.

*Memory trick:* etc = settings, var = logs and changing data, home = my stuff, tmp = throwaway, usr = programs.

---

## 3. Paths, hidden files, wildcards and quoting

### Absolute vs relative paths

- **Absolute path:** starts with `/`. It works from anywhere. Example: `/var/log/syslog`
- **Relative path:** does *not* start with `/`. It starts from where you are now (`pwd`). Example: `log/syslog` only works if you are inside `/var`.

| Symbol | Meaning |
|---|---|
| `.` | The current directory |
| `..` | The parent directory (one level up) |
| `~` | Your home directory |
| `-` | The previous directory (`cd -` jumps back) |

```bash
cd /var/log      # absolute
cd ..            # relative: go up to /var
cd ~             # go home
cd -             # back to the last place
```

**Linux is case-sensitive:** `Notes.txt` and `notes.txt` are two different files.

### Hidden files

Any file or folder whose name **starts with a dot** is hidden. Plain `ls` will not show it. Use `ls -a`. Examples in your home: `.bashrc` (shell settings), `.ssh/` (SSH keys), `.profile`.

### Wildcards (the shell expands these before the command runs)

| Pattern | Matches | Example |
|---|---|---|
| `*` | Any number of characters | `ls *.txt` |
| `?` | Exactly one character | `ls file?.txt` |
| `[abc]` | One character from the set | `ls file[12].txt` |
| `[0-9]` | One character in the range | `ls log[0-9].txt` |

### Quoting basics

| Quote | Behaviour | Example |
|---|---|---|
| `'single'` | Everything is literal | `echo '$HOME'` prints `$HOME` |
| `"double"` | Variables still work, wildcards do not | `echo "$HOME"` prints `/home/pav` |
| `\` | Escapes the next character | `ls My\ Files` |

When a name has **spaces**, always quote it: `cd "My Documents"`.

---

## 4. Getting help and finding commands

| Command | What it does |
|---|---|
| `man ls` | Full manual. Keys: `Space` next page, `/word` search, `n` next match, `q` quit |
| `ls --help` | Short usage summary (quick and handy) |
| `type ls` | Tells you what kind of command it is: alias, builtin, or file |
| `which python3` | Shows the full path of the program that would run |
| `history` | Lists the commands you typed before |

**Time-saving keys:**

- `Tab`: autocomplete names. Press twice to list options.
- `Up` / `Down`: scroll through previous commands.
- `Ctrl+R`: search your command history.
- `!!`: repeat the last command (`sudo !!` repeats it with sudo).
- `Ctrl+C`: stop a running command. `Ctrl+L`: clear the screen.

---

## 5. Identity and system information

| Command | What it shows |
|---|---|
| `whoami` | Your username |
| `id` | Your user ID (uid), group ID (gid) and all groups |
| `hostname` | The name of this machine |
| `uname -a` | Kernel name, version and architecture (`uname -r` for just the kernel version) |
| `date` | Current date and time |
| `uptime` | How long it has been running, users logged in, and **load average** (the last 1, 5 and 15 minutes) |

---

## 6. Command cheat sheet

| Command | Purpose | Try this |
|---|---|---|
| `pwd` | Print working directory: where am I? | `pwd` |
| `ls -lah` | List files: **l**ong format, **a**ll (including hidden), **h**uman-readable sizes | `ls -lah ~` |
| `cd` | Change directory | `cd /etc` |
| `tree` | Show folders as a tree (install: `sudo apt install tree`) | `tree -L 1 /` |
| `man` | Open the manual | `man ls` |
| `type` / `which` | Find out what a command is and where it lives | `type cd` |
| `history` | Show past commands | `history \| tail` |
| `whoami`, `id` | Who am I? | `id` |
| `hostname`, `uname -a` | Which machine and kernel? | `uname -a` |
| `date`, `uptime` | Time and running time | `uptime` |

### How to read `ls -lah` output

```
-rw-r--r--  1 pav pav  220 Oct 17 10:00 .bash_logout
drwxr-xr-x  2 pav pav 4.0K Oct 17 10:05 Documents
```

| Part | Meaning |
|---|---|
| First character | `-` file, `d` directory, `l` symbolic link |
| Next 9 characters | Permissions: owner, group, others (covered on Day 3) |
| `pav pav` | Owner and group |
| `220`, `4.0K` | Size |
| Date, then name | Last modified and the name |

---

## 7. Hands-on lab

Type each step yourself and write down what you see.

**Step 1: Where am I?**
```bash
pwd
whoami
hostname
```

**Step 2: Visit /etc**
```bash
cd /etc
pwd
ls
ls -lah | head -20
```

**Step 3: Visit /var/log**
```bash
cd /var/log
ls -lah
```
If a file says *Permission denied* when you try to read it, that is normal. Note it down; permissions come on Day 3.

**Step 4: Go home and list hidden files**
```bash
cd ~
ls -a
ls -lah
```
Find `.bashrc` in the list.

**Step 5: Find help for ls**
```bash
ls --help | less
man ls
```
Find out what these flags do: `-R`, `-t`, `-S`, `-r`. (Quit with `q`.)

**Step 6: Practise paths**
```bash
cd /var/log
cd ..
pwd
cd -
cd ~
ls /etc/*.conf
```

**Step 7: System information**
```bash
id
uname -a
date
uptime
cat /etc/os-release
```

**Step 8: Look at your history**
```bash
history | tail -20
```

### The purpose of five key directories

| Directory | Purpose |
|---|---|
| `/etc` | System configuration |
| `/var/log` | Log files |
| `/home` | Users' personal files |
| `/tmp` | Temporary files |
| `/usr` | Installed programs and libraries |

---

## 8. Interview self-check

**Q1. What is the difference between the Linux kernel and a shell?**
The kernel is the core of the operating system. It controls hardware, memory, processes and files. The shell is a user-space program that takes the commands you type and asks the kernel to run them. The shell is the interface; the kernel does the real work.

**Q2. What is the difference between an absolute and a relative path?**
An absolute path starts at the root `/` and works from anywhere (`/var/log/syslog`). A relative path starts from the current directory and depends on where you are (`../log/syslog`).

**Bonus questions to answer out loud:**

- What is the difference between a terminal and a shell?
- What does `/etc` contain? What about `/var`?
- What does `ls -lah` do, flag by flag?
- How do you see hidden files?
- What is the difference between `'$HOME'` and `"$HOME"`?
- What does `type` tell you that `which` does not?

---

## 9. Common errors and fixes

| Message | Usual cause | Fix |
|---|---|---|
| `No such file or directory` | Typo, wrong path, or wrong capital letters | Check `pwd` and `ls`; use `Tab` to complete |
| `Permission denied` | You are not allowed to read or write there | Normal for many system files; do not reach for `sudo` blindly |
| `command not found` | Typo, or the program is not installed | Check spelling; `sudo apt install <name>` |
| `cd: too many arguments` | A folder name has a space | Quote it: `cd "My Folder"` |

### My error log (fill in as you go)

| Command or error | What I did | The fix |
|---|---|---|
| | | |
| | | |
| | | |

---

## 10. End-of-day sign-off

- [ ] I can explain today's topics without reading the notes.
- [ ] I completed the lab and can repeat it independently.
- [ ] I recorded unfamiliar commands, errors and fixes.
