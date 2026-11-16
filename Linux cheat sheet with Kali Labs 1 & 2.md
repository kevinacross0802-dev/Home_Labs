# Linux Cheat Sheet
*The commands you've learned so far, in one place.*

## Where am I? (navigation)

| Command | What it does |
|---|---|
| `pwd` | Print working directory — shows where you're standing |
| `ls` | List what's in the current directory |
| `ls foldername` | Peek inside a folder without going in |
| `cd foldername` | Move into a folder |
| `cd ..` | Move back out to the parent folder (one level up) |
| `cd ~` | Jump straight home from anywhere |
| `cd` | Same as `cd ~` — bare `cd` goes home |

Your prompt tells you where you are: the part after the `:` is your current folder.
`~` = home, `/` = the root (very top of the whole system).

## Paths

- A **relative path** starts from where you're standing: `worldbanc/public`
- An **absolute path** starts from the root and works anywhere: `/home/user/worldbanc/public`
- `/` separates folders — never dots or backslashes (that's Windows)
- `..` = the folder one level up

## Looking at files

| Command | What it does |
|---|---|
| `cat file.txt` | Print the whole file |
| `head file.txt` | Print the first 10 lines |
| `head -n 6 file.txt` | Print the first 6 lines |
| `tail file.txt` | Print the last 10 lines |
| `tail -n 5 file.txt` | Print the last 5 lines |

## Searching inside files

| Command | What it does |
|---|---|
| `grep "CRITICAL" file.log` | Print only the lines in that file containing `CRITICAL` |
| `grep -r "CRITICAL" folder` | Search every file in a folder (and folders inside it) for `CRITICAL` |

`grep` takes the pattern first, then where to look: `grep "what" where`. With more than one file searched, each result starts with the filename it came from. `grep` is case-sensitive — `"CRITICAL"` only matches the all-caps version.

## Creating things

| Command | What it does |
|---|---|
| `touch newfile.txt` | Create an empty file (or update its timestamp if it exists) |
| `mkdir newfolder` | Create a new directory |

## Moving, copying, deleting

| Command | What it does |
|---|---|
| `mv old/path/file.txt new/path/` | Move a file into another folder (name stays the same if the destination is a folder ending in `/`) |
| `mv file.txt newname.txt` | Rename a file (same command — moving to a *filename* renames it) |
| `cp file.txt folder/` | Copy a file — the original stays put |
| `cp -R folder newfolder` | Copy a whole directory and everything in it |
| `rm file.txt` | Delete a file — **no trash, no undo** |
| `rm -r folder` | Delete a directory and everything inside it |

Always `ls` the exact path before an `rm` — check twice, delete once.

Silence = success. `touch` and `mkdir` print nothing when they work.

## Finding things

| Command | What it does |
|---|---|
| `find ~ -name "2023.csv"` | Search your whole home directory for a file by name and print its full path |
| `find public/products -name "*joint*"` | Search `public/products` for filenames containing `joint` |

The first thing after `find` is **where to search**, and `-name` says what pattern to match. `*` is a wildcard — it matches anything — so `"*joint*"` means "joint somewhere in the name" and `"*.txt"` means anything ending in `.txt`. Keep the quotes around the pattern so the shell doesn't try to expand the `*` itself.

## The map of the filesystem

| Place | What's in it |
|---|---|
| `/` | The root — the top of everything |
| `/home/user` (your `~`) | Your personal files live here |
| `/etc` | Configuration files |
| `/var` | Variable data that changes as the system runs — logs in `/var/log`, print queues in `/var/spool` |
| `/boot` | The kernel and bootloader files |
| `/usr` | Programs and their libraries (e.g. `/usr/lib`) |

## Checking the machine

| Command | What it does |
|---|---|
| `free -h` | Show RAM — total, used, and free, in human-readable sizes (Gi/Mi) |
| `df -h` | Show disk space — how much is used and free on each partition |

On the Surface Pro 4 (2026-11-11): 3.8Gi RAM total, ~957Mi free with Kubuntu running — so one light VM at a time. Disk is fine: 45G free on `/`, 37G free on `/home`. The first VM was planned at 1GB RAM, but the Ubuntu live installer kernel-panicked at 1GB — it needs 2GB just to unpack itself (the installed server can run on less later).

## Installing software

| Command | What it does |
|---|---|
| `sudo apt update` | Refresh the list of available packages before installing |
| `sudo apt install -y package-name` | Install a package (`-y` answers yes to the prompt). `sudo` asks for your password — nothing shows on screen while you type it, that's normal |

## Home lab — virtualization (Surface Pro 4, 2026-11-11)

| Command | What it does |
|---|---|
| `egrep -c '(vmx|svm)' /proc/cpuinfo` | Count CPU cores reporting virtualization support — anything above 0 means it works (the Surface printed 8) |
| `sudo apt install -y qemu-system-x86 libvirt-daemon-system libvirt-clients bridge-utils virt-manager` | Install KVM + the VM manager app. Note the exact names: `qemu-kvm` doesn't exist as a real package on this Ubuntu version (it's a "virtual package") — the real one is `qemu-system-x86` — and it's `libvirt-clients`, spelled with an `i` after the `v` |
| `sudo adduser kevinc libvirt` | Add your user to the `libvirt` group, so you can run VMs |
| `sudo adduser kevinc kvm` | Add your user to the `kvm` group, same reason |
| `virsh list --all` | List VMs — an empty table with no error means the install works. Run after rebooting, because group membership only takes effect at login |
| `virsh setmaxmem ubuntu24.04 2048M --config` | Raise a VM's maximum memory to 2048 MB (VM must be off; `--config` saves it). Stick the `M` to the number — `setmaxmem` does **not** accept a `--unit` flag, that errors out |
| `virsh setmem ubuntu24.04 2048M --config` | Raise a VM's current memory to 2048 MB, same rules as above |
| `virsh dominfo ubuntu24.04 \| grep -i memory` | Check a VM's memory in writing — Max memory and Used memory should both read `2097152 KiB` (that's 2048 MB) |
| `hostname -I` | Inside the VM, print its IP address — the server printed `192.168.122.210` |
| `virsh snapshot-create-as ubuntu24.04 fresh-install` | Save a rewind point called `fresh-install` for the whole VM — run this **on the Surface**, not inside the VM |
| `virsh snapshot-list ubuntu24.04` | Check the snapshot landed — `fresh-install` should be listed with today's date |

First VM (2026-11-11): downloaded from ubuntu.com/download/server — the file that arrived was **Ubuntu 26.04.1** (`ubuntu-26.04.1-live-server-amd64.iso`), newer than the planned 24.04. In Virtual Machine Manager, auto-detect didn't recognise 26.04, so pick the newest Ubuntu in the list by hand. The VM itself was named `ubuntu24.04` even though the ISO is 26.04 — the name is just a label, it doesn't have to match the ISO. Planned size: 1 CPU, 2048 MB RAM (after the 1GB panic), 10 GiB disk, NAT network. Before booting a 2GB VM on a 4GB host, close Chrome and other apps so the RAM is actually free. Inside the VM console, the mouse gets grabbed — **Ctrl+Alt** releases it back to the Surface.

Boot order and media (2026-11-11, all run into and fixed the same afternoon): the first boot after the memory fix said "No bootable device" because only **VirtIO Disk 1** was ticked under Boot device order — fix in the VM's details ("i" icon) → **Boot Options**: tick **SATA CDROM 1**, move it **above** VirtIO Disk 1 with the ▲ arrow, Apply. The next boot said "Could not read from CDROM (code 0003)" because the drive was empty — details → **SATA CDROM 1** showed Source path "No media selected" — fix on that same page: **Browse** → pick `ubuntu-26.04.1-live-server-amd64.iso` (Browse Local → Downloads if needed), Apply. Installer screens after that: accept the defaults, tick **Install OpenSSH server** on the SSH Setup screen, and accept the storage defaults (it only touches the VM's own 10GB disk). **Before rebooting at "Installation complete", eject the virtual DVD first** — details → SATA CDROM 1 → clear the Source path (✕) back to "No media selected" → Apply — then click **[ Reboot Now ]**. That button reboots only the VM, not the Surface. First boot prints SSH host keys and `cloud-init ... finished`; press Enter once for a clean `ubuntu-server login:` prompt, then log in as `kevinc` with the password set in the installer. Install completed 2026-11-11, logged in, IP `192.168.122.210`, `fresh-install` snapshot taken. SSH confirmed working 2026-11-12: from a Surface terminal, `ssh kevinc@192.168.122.210` (type `yes` at the fingerprint question, then the server password) — the prompt changes from `kevinc@kevin-surfacepro4` to `kevinc@ubuntu-server`, and `exit` brings you back to the Surface. First attempt got "Connection refused" only because the VM was still booting and its SSH service wasn't up yet; the second attempt connected. That's how the lab is used day to day, instead of the console window. The login banner reported 45 updates waiting (`sudo apt update && sudo apt upgrade -y`); whether that upgrade finished is not yet confirmed.

Second VM — Kali (2026-11-11): ISO `kali-linux-2026.3-installer-amd64.iso` (Downloads; double-clicking it opens Ark — close Ark, don't extract anything). In Virtual Machine Manager: Local install media → Browse Local → that ISO. The wizard has no Kali entry, so pick the **newest Debian** (Kali is built on Debian, so the Debian defaults are correct). Settings: **2048 MB RAM, 2 CPUs, 20 GB disk, name `kali`**, NAT. Installer: **Graphical install** → English / United States / American English → hostname `kali`, domain blank → user: keep the installer's suggested name — on this build it was **`kevinc`**, NOT `kali` (there is **no user named `kali`** on the VM; `passwd kali` answers "user 'kali' does not exist", and every `kali` login fails with "Login incorrect" even with the right password) → timezone Eastern → **Guided – use entire disk** → **All files in one partition** → Finish partitioning, Yes → leave **Software selection exactly as it is** → GRUB Yes, pick the VM's disk (`/dev/vda…` or `/dev/sda…`). Install finished 2026-11-11, faster than the 30–45 minute estimate. Same eject-before-reboot drill as Ubuntu: details → **SATA CDROM 1** → clear Source path to "No media selected" → Apply → **Continue**. **Registered name is `debian13`, not `kali`** — the wizard kept the Debian profile name — under the virt-manager connection **`cyber_security`**. Login resolved the same evening: catch GRUB by holding the **Down arrow**, boot with `init=/bin/bash`, then `mount -o remount,rw /`, `ls /home` (prints the real username — `kevinc`), `passwd kevinc`, reboot, log in as `kevinc`. Snapshot: `virsh snapshot-create-as kali fresh-install` fails with "failed to get domain 'kali'" — `virsh list --all` on the Surface shows only `ubuntu24.04` and `virsh -c qemu:///session list --all` is empty, while virt-manager plainly lists both VMs under `cyber_security` (why the lists disagree is unresolved; the manager is the ground truth). So take Kali's snapshot in the **GUI**: double-click `debian13` → the far-right toolbar icon ("Manage VM snapshots") → **`+`** → Name `fresh-install` → Finish. Done 2026-11-11 18:54:28, VM Shutoff. Kali build closed. Cleanup queued for a later lab session: rename `debian13` → `kali`, and work out why `virsh`'s lists disagree with the manager.

## Gotchas that bite beginners

1. **"No such file or directory" almost always means you're in the wrong folder**, not that the file is gone. Check where you're standing with `pwd` and `ls`.
2. **Linux is case-sensitive.** `report.txt` and `Report.txt` are two different files. Match capitals exactly.
3. **Spaces matter.** `cd ..` works; `cd..` is read as one unknown command and fails.
4. A bare filename only works for files in your **current** directory. For anything else, give the path: `cat worldbanc/public/pr_ideas.txt`.
5. The BIOS runs the POST and initializes hardware at power-on, then hands off to the bootloader (GRUB), which loads the kernel. `init`/`systemd` (PID 1) is the parent of all processes after that.
6. Distro families: **Debian** uses DPKG/APT, **Red Hat** uses RPM with `yum`/`dnf` (Fedora is its upstream testing ground), **SUSE** has YaST.
7. **Don't copy a folder into a folder inside itself.** `backups` lives inside `transactions`, so `cp -R transactions backups/` would try to copy the folder into itself, forever. Copy the single missing file instead — `cp 2020.csv backups/`.
8. **`grep` needs two parts: what, then where.** `grep worldbanc/private/logs/2024-01-10.log` fails because there's nothing to search *for* — it needs `grep "CRITICAL" worldbanc/private/logs/2024-01-10.log`.
9. **The space between a command and its flag is part of the command.** `df-h` was read as one made-up command and failed; `df -h` works. Same rule as `cd ..` in gotcha 3 — and package names are just as exact: `libvirt-clients` with a typo installs nothing, apt just stops.
10. **`virsh setmaxmem` doesn't take a `--unit` flag** — `virsh setmaxmem ubuntu24.04 2048 --config --unit M` errors out. Write the size as one piece instead: `2048M`. Like `touch`, success is silence: no message, just a fresh prompt. And `virsh` is v-i-r-s-h — `vrish` (letters swapped) is not a command.
11. **Your prompt tells you which machine you're on — read it before running a command.** A prompt ending in `ubuntu-server` is *inside* the VM; one ending in `kevin-surfacepro4` is the Surface itself. `virsh` only exists on the Surface — running it inside the VM just says "command not found", because a VM can't snapshot itself.
12. **Eject the installer DVD before rebooting, or the install starts over.** With the CDROM still first in the boot order and the ISO still loaded, Reboot just boots the installer again — clear the CDROM source path first (see "Boot order and media" above).
13. **"Login incorrect" can mean the username doesn't exist, not that the password is wrong.** On Kali, `passwd kali` answered "user 'kali' does not exist" — the real account was `kevinc`. Before resetting a password, check who actually lives there: from a recovery shell, `ls /home` prints the real username(s).
14. **nmap flags are case-sensitive.** `-Pn -F` works; lowercase `-pn -f` fails — nmap reads `-p` as "ports:" and chokes on the `n`.

## First nmap scan — Kali Lab 1 (2026-11-13)

Lab network that morning: Kali `eth0` at **192.168.122.96/24**, Ubuntu Server at **192.168.122.210** — same 192.168.122 street. Kali is the VM named `debian13` under the `cyber_security` connection; log in as `kevinc`, open a terminal with **Ctrl+Alt+T** (prompt reads `(kevinc@kali)-[~]`).

| Command | What it does |
|---|---|
| `ip -br a` | Show your own addresses, briefly — `eth0` is your address on the lab network |
| `nmap scanme.nmap.org` | Scan a target's 1,000 most common ports. First run took **562.26 seconds** — slow because filtered ports stay silent and nmap waits and retries |
| `nmap -Pn -F scanme.nmap.org` | The fast version: `-F` checks only the top 100 ports, `-Pn` skips the "are you there?" ping phase NAT likes to stall on. Same open doors, in **3.12 seconds** |

How to read a report: **open** means the door answered; **closed** means it answered "no" instantly (a reset), so it costs nothing; **filtered** means silence — a firewall in front, not open and not closed, and silence is what makes scans slow.

What Lab 1 found on scanme.nmap.org (45.33.32.156): **22/tcp open (ssh)**, **80/tcp open (http)**, **9929/tcp open (nping-echo)** — Nmap's own test service, left open on purpose; **25, 135, 139, 445 filtered**. The fast scan found the same 22 + 80, plus "94 closed tcp ports (reset)" — 9929 didn't appear because it's not in the top 100, which is correct, not a miss.

The ethics rule, from Fyodor's own page on scanme.nmap.org: you are authorized to scan this machine, a few scans a day is fine, don't hammer it, and don't practice password-cracking on it. That permission is what makes it a practice target.

Shut Kali down cleanly from its terminal with `sudo poweroff` (it asks for your password; typing shows nothing — normal).

## Scanning your own server — Kali Lab 2 (2026-11-15)

Target: his own Ubuntu Server at **192.168.122.210**, with both Surface VMs awake (server booted, Kali already up). Kali's own address re-checked first with `ip a`: **192.168.122.96/24** on eth0, still current.

| Command | What it does |
|---|---|
| `nmap -Pn -F 192.168.122.210` | Fast top-100 scan of the server. Result: **22/tcp open ssh**, "Not shown: 99 closed tcp ports (reset)", MAC shown as a QEMU virtual NIC, done in **0.58 seconds**. From outside, the server shows exactly one door — the SSH he installed on purpose |
| `ssh kevinc@192.168.122.210` | Walk through that door: type `yes` at the fingerprint prompt (the server's ID card — Kali remembers it and will warn if it ever changes), then the server password. The prompt changes from `(kevinc@kali)` to `kevinc@ubuntu-server` — every command now runs **on the server** |
| `hostname` | Proves which machine is answering: `ubuntu-server` |
| `exit` | Logs out of SSH and returns to the Kali prompt |

The lesson of the lab: the scan is just knocking on doors — recon. What separates *hacking* from *administering* is the key: he got in with his own password, on his own machine, with his own permission. Without the password, an attacker is still standing outside that one locked door — and the server logs the failed knock (which is why defenders watch SSH logs).

The server's login banner that day reported **6 updates can be applied immediately, all 6 of them standard security updates** — applied the same evening while logged in (`sudo apt update`, then `sudo apt upgrade -y`). Clean finish: needrestart reported **"Running kernel seems to be up-to-date"** (no reboot needed), restarted the updated services (including ssh — the live session survived), and deferred a few restarts (dbus, systemd-logind, getty) — deferred is normal, they pick up on the next boot. "User sessions running outdated binaries" just means your current login is still holding pre-update programs in memory; logging out and back in clears it.
