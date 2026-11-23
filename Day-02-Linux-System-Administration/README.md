# Linux System Administration — Day 02

*Ubuntu Server, filesystem, users, permissions, processes and services*

## 1. Linux basics

Linux is an operating-system kernel used by distributions such as Ubuntu, Debian, Fedora and Kali Linux. A distribution combines the kernel with system tools, libraries, package management and applications.

These notes use Ubuntu Server. Most commands also work on other distributions, although package-management and service names can differ.

- **Kernel:** Core of the operating system; manages CPU time, memory, devices and communication between hardware and software.
- **Shell:** Command interpreter used to interact with the operating system. Bash is common on Linux.
- **Terminal:** The interface in which a shell runs.
- **Root:** The administrative account with user ID 0.
- **Distribution:** A packaged Linux operating system, such as Ubuntu Server.

Check the current account and system:

~~~bash
whoami
id
hostname
uname -a
cat /etc/os-release
~~~

## 2. Linux filesystem hierarchy

Linux uses one directory tree. Its top is /, called the root directory. Other directories and mounted storage appear below it.

| Directory | Purpose |
|---|---|
| / | Top of the filesystem hierarchy |
| /home | Home directories for ordinary users |
| /root | Home directory of the root account |
| /etc | System-wide configuration files |
| /var | Variable data such as logs, caches and application data |
| /var/log | System and application log files |
| /tmp | Temporary files |
| /usr | User-space programs, libraries and shared data |
| /bin | Essential user commands; commonly linked to /usr/bin on modern Ubuntu |
| /sbin | System administration commands; commonly linked to /usr/sbin |
| /boot | Bootloader files and Linux kernel images |
| /dev | Device files representing hardware and virtual devices |
| /proc | Virtual filesystem exposing process and kernel information |
| /sys | Virtual filesystem exposing devices and kernel information |
| /mnt | Common temporary mount point for filesystems |
| /media | Common mount location for removable media |
| /opt | Optional third-party application software |

### 2.1 Navigation

~~~bash
pwd
ls
ls -l
ls -la
cd /etc
cd ..
cd ~
cd /
~~~

- **pwd:** prints the current working directory.
- **ls:** lists directory contents.
- **ls -l:** shows permissions, owner, group and size.
- **ls -a:** includes hidden entries whose names begin with a dot.
- **cd ..:** moves to the parent directory.
- **cd ~:** moves to the current user's home directory.
- **cd /**: moves to the filesystem root.

### 2.2 Absolute and relative paths

An **absolute path** starts from /, for example /home/alen/Documents.

A **relative path** is interpreted from the current directory, for example Documents/report.txt.

- / — filesystem root.
- ~ — current user's home directory.
- . — current directory.
- .. — parent directory.

### 2.3 File and directory operations

~~~bash
mkdir practice
mkdir -p practice/logs/archive
touch practice/notes.txt
cp practice/notes.txt practice/notes-copy.txt
mv practice/notes-copy.txt practice/backup.txt
cat practice/notes.txt
less /etc/os-release
head /var/log/syslog
tail /var/log/syslog
tail -f /var/log/syslog
~~~

Remove files and directories carefully:

~~~bash
rm practice/backup.txt
rmdir empty-directory
rm -r practice
~~~

**Important:** rm normally deletes files without moving them to a recycle bin. Check the path before using rm -r, and avoid broad paths or unnecessary root privileges.

## 3. Users and groups

Linux is a multi-user operating system. Each process runs with a user identity and group memberships. These identities are used when checking access to files and system resources.

### 3.1 User information

~~~bash
whoami
id
who
w
getent passwd
getent group
~~~

Important files:

| File | Purpose |
|---|---|
| /etc/passwd | User account records and basic account information |
| /etc/shadow | Password hashes and password-aging information; normally restricted |
| /etc/group | Group records and supplementary group membership |
| /etc/sudoers | Sudo authorization rules |

Do not edit /etc/shadow or /etc/sudoers casually. Use the appropriate administration commands.

### 3.2 Creating and managing users

Create a user with a home directory and interactive setup:

~~~bash
sudo adduser testuser
~~~

Set or change a password:

~~~bash
sudo passwd testuser
~~~

Inspect the account:

~~~bash
id testuser
getent passwd testuser
~~~

Lock and unlock an account:

~~~bash
sudo passwd -l testuser
sudo passwd -u testuser
~~~

Delete a test account:

~~~bash
sudo deluser testuser
~~~

Delete the account and its home directory:

~~~bash
sudo deluser --remove-home testuser
~~~

Only use account deletion commands on accounts you are authorized to remove. Confirm whether the user's files must be retained first.

### 3.3 Groups

Groups make it easier to grant the same access to multiple users.

~~~bash
sudo groupadd support
sudo usermod -aG support testuser
groups testuser
id testuser
getent group support
sudo gpasswd -d testuser support
sudo groupdel support
~~~

The -aG options append a supplementary group to the user's existing memberships. Omitting -a when using -G can replace the existing supplementary group list.

### 3.4 sudo and root privileges

The sudo command allows an authorized user to run a command with another user's privileges, commonly root.

~~~bash
sudo apt update
sudo systemctl status ssh
sudo -i
~~~

- **sudo command:** runs one command with elevated privileges.
- **sudo -i:** opens a root login shell. Exit it with exit.

On Ubuntu, users created with adduser are not automatically administrators. An authorized administrator can add a user to the sudo group:

~~~bash
sudo usermod -aG sudo testuser
~~~

The user may need to sign out and sign in again for the new group membership to apply.

**Security practice:** Use a normal account for everyday work and use sudo only when an administrative task requires it. Do not share the root password.

## 4. File permissions and ownership

Linux permissions control who can read, modify or execute a file or directory.

Run ls -l to inspect permissions. Example:

~~~text
-rwxr-xr-- 1 alen support 1200 Nov 23 10:00 script.sh
~~~

The first character identifies the file type. The next nine characters are permissions, grouped into three sets:

- Owner: rwx
- Group: r-x
- Others: r--

| Permission | File meaning | Directory meaning |
|---|---|---|
| r | Read file contents | List directory entries |
| w | Modify file contents | Create, remove or rename entries, subject to other controls |
| x | Execute the file | Enter or traverse the directory |

### 4.1 Numeric permissions

Each permission has a numeric value:

| Permission | Value |
|---|---:|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |

Add the values for each owner/group/others permission set:

- **7** = 4 + 2 + 1 = read, write and execute.
- **6** = 4 + 2 = read and write.
- **5** = 4 + 1 = read and execute.
- **4** = read only.

For example, 755 means owner has rwx, while group and others have r-x.

~~~bash
chmod 755 script.sh
chmod 644 notes.txt
chmod u+x script.sh
chmod g-w shared.txt
~~~

### 4.2 Ownership

~~~bash
sudo chown testuser notes.txt
sudo chown testuser:support notes.txt
sudo chgrp support notes.txt
~~~

Apply ownership changes recursively only when intended:

~~~bash
sudo chown -R testuser:support /path/to/directory
~~~

Recursive permission or ownership changes can affect many files. Verify the target path before running them.

### 4.3 Special permissions

- **SUID:** An executable runs with the file owner's effective user ID.
- **SGID:** On executable files, it affects the effective group used when running the program. On directories, new entries commonly inherit the directory's group.
- **Sticky bit:** On shared directories, users generally can remove or rename only files they own (or when they have appropriate privileges).

Inspect permissions:

~~~bash
ls -l /usr/bin/passwd
ls -ld /tmp
find / -perm -4000 -type f 2>/dev/null
~~~

The last command searches for SUID files. The presence of an SUID bit is not automatically evidence of compromise.

### 4.4 ACLs — Access Control Lists

Traditional permissions provide owner, group and others access. ACLs allow additional named users or groups to receive specific permissions.

Install ACL tools if needed:

~~~bash
sudo apt install acl
~~~

Set an ACL for a user, view it and remove it:

~~~bash
sudo setfacl -m u:testuser:rw notes.txt
getfacl notes.txt
sudo setfacl -x u:testuser notes.txt
~~~

ACLs are useful when one user needs access without changing the file's primary owner or group.

## 5. Processes and resource usage

A process is a running instance of a program. Linux assigns each process a Process ID (PID).

~~~bash
ps
ps aux
ps -ef
top
pgrep ssh
pidof sshd
ps -p 1 -f
free -h
uptime
~~~

- **ps aux:** displays processes from all users in a detailed format.
- **top:** shows live process and resource information.
- **pgrep:** finds process IDs by name or pattern.
- **free -h:** displays memory usage in human-readable units.
- **uptime:** shows how long the system has been running and load averages.

Stop a process gracefully:

~~~bash
kill PID
~~~

If a process does not stop and you understand the impact, a forceful signal is available:

~~~bash
kill -9 PID
~~~

Replace PID with the actual process ID. Prefer normal termination first. Do not kill system processes just because their names are unfamiliar.

## 6. Services and systemd

A service is a background program that provides a system or application function. Ubuntu Server uses **systemd** as its service manager.

~~~bash
systemctl status ssh
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl enable ssh
sudo systemctl disable ssh
~~~

- **status:** displays service state and recent messages.
- **start:** starts a service now.
- **stop:** stops a service.
- **restart:** restarts a service.
- **enable:** configures a service to start automatically at boot.
- **disable:** removes its automatic-start configuration.

List active services and check boot configuration:

~~~bash
systemctl list-units --type=service --state=running
systemctl is-enabled ssh
~~~

A service can be running but not enabled at boot, or enabled but currently stopped. These are different states.

### 6.1 Viewing logs with journalctl

The systemd journal stores logs from the kernel and services.

~~~bash
journalctl
journalctl -b
journalctl -u ssh
journalctl -u ssh --since today
journalctl -p err
journalctl -f
~~~

- **journalctl -b:** logs from the current boot.
- **journalctl -u ssh:** messages for the SSH service.
- **--since today:** limits output to today's entries.
- **-p err:** shows error-priority messages.
- **-f:** follows new log entries.

Use logs together with service status when troubleshooting.

## 7. SSH remote administration

SSH (Secure Shell) provides encrypted remote command-line access to a Linux machine.

On Ubuntu Server, install the OpenSSH server if it is not already installed:

~~~bash
sudo apt update
sudo apt install openssh-server
~~~

Check the service and find the server's IP address:

~~~bash
sudo systemctl status ssh
ip address
hostname -I
~~~

Connect from another machine:

~~~bash
ssh username@SERVER_IP
~~~

For example:

~~~bash
ssh alen@192.168.1.50
~~~

The first connection may ask you to verify the server's host key fingerprint. Verify it through a trusted channel before accepting it, especially in a production environment.

### 7.1 SSH configuration

The server configuration is commonly stored in /etc/ssh/sshd_config.

After changing SSH server configuration, validate it and reload the service:

~~~bash
sudo sshd -t
sudo systemctl reload ssh
~~~

Do not disable password authentication or root login until key-based access has been tested and you have a working recovery path. A bad SSH configuration can lock you out of a remote server.

Basic SSH security:
- Use individual user accounts rather than sharing credentials.
- Prefer SSH keys for regular administration.
- Keep the server updated.
- Restrict access to trusted networks where practical.
- Review authentication logs for unexpected attempts.
- Avoid exposing SSH directly to the internet unless the access requirements and protections are understood.

## 8. Disk usage and mounting

Linux makes storage devices available by mounting their filesystems at directories called mount points.

~~~bash
lsblk
lsblk -f
df -h
du -sh /var/log
findmnt
mount
~~~

- **lsblk:** lists block devices and partitions.
- **lsblk -f:** also shows filesystem type, label and UUID.
- **df -h:** shows filesystem capacity and free space.
- **du -sh:** estimates space used by a directory.
- **findmnt:** displays mounted filesystems and mount points.

### 8.1 Mounting a filesystem

A mount point is an existing directory where a filesystem becomes accessible.

Example only — replace the device with the correct one from lsblk:

~~~bash
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
findmnt /mnt/data
~~~

Unmount it:

~~~bash
sudo umount /mnt/data
~~~

Do not assume a disk is always named /dev/sdb1. Device names can change. Confirm the correct device and filesystem before mounting or formatting anything.

### 8.2 Persistent mounts and /etc/fstab

The /etc/fstab file describes filesystems that should be mounted automatically.

Find the filesystem UUID:

~~~bash
sudo blkid
~~~

An example entry has this structure:

~~~text
UUID=your-filesystem-uuid  /mnt/data  ext4  defaults  0  2
~~~

The UUID, mount point, filesystem type and options must match the actual disk. Do not copy the placeholder UUID.

After editing /etc/fstab, test the configuration:

~~~bash
sudo mount -a
findmnt --verify
~~~

Keep a backup of the original file and ensure you can access the machine if a mount configuration prevents normal boot.

## 9. Package management with APT

APT is the package-management tool used by Ubuntu and Debian-based systems.

~~~bash
sudo apt update
sudo apt upgrade
sudo apt install nginx
sudo apt remove nginx
sudo apt purge nginx
apt search nginx
apt show nginx
sudo apt autoremove
~~~

- **apt update** refreshes package lists; it does not itself upgrade installed packages.
- **apt upgrade** upgrades installed packages.
- **apt install** installs a package.
- **apt remove** removes a package but may leave configuration files.
- **apt purge** removes a package and its system configuration files.
- **apt autoremove** removes packages that are no longer required.

Review upgrade changes before confirming them on a production server.

## 10. Scheduled tasks with cron

Cron runs commands or scripts on a schedule.

~~~bash
crontab -e
crontab -l
~~~

A cron entry has five time fields followed by the command:

~~~text
minute hour day-of-month month day-of-week command
~~~

Example — run a script every day at 2:30 AM:

~~~text
30 2 * * * /home/alen/scripts/backup.sh
~~~

| Field | Meaning | Example |
|---|---|---|
| Minute | 0–59 | 30 |
| Hour | 0–23 | 2 |
| Day of month | 1–31 | * |
| Month | 1–12 | * |
| Day of week | 0–7 (Sunday is 0 or 7) | * |

Special characters:
- ***** — every possible value.
- **,** — list of values.
- **-** — range of values.
- **/** — step value.

Cron jobs run with a limited environment. Use full paths to commands and files, and redirect output to a log when troubleshooting.

## 11. Basic Linux troubleshooting

### 11.1 Check system load and memory

~~~bash
uptime
top
free -h
df -h
~~~

Check whether a filesystem is full before assuming an application problem.

### 11.2 Check network configuration

~~~bash
ip address
ip route
resolvectl status
ping -c 4 8.8.8.8
getent hosts ubuntu.com
~~~

- **ip address:** displays network interfaces and IP addresses.
- **ip route:** displays routing information.
- **resolvectl status:** shows DNS resolver configuration on systems using systemd-resolved.
- **ping:** tests basic IP reachability.
- **getent hosts:** checks hostname resolution using configured name-service sources.

A failed ping does not always mean a host is unavailable; ICMP may be blocked. Compare multiple checks.

### 11.3 Check service failures

~~~bash
systemctl --failed
systemctl status SERVICE_NAME
journalctl -u SERVICE_NAME -n 50
~~~

Replace SERVICE_NAME with the actual service name.

### 11.4 Check authentication activity

On Ubuntu, authentication messages are commonly available through the system journal. Depending on configuration, they may also be written to /var/log/auth.log.

~~~bash
sudo journalctl -u ssh
sudo journalctl -p warning
sudo tail -n 50 /var/log/auth.log
~~~

If a log file does not exist, check the journal and the machine's logging configuration rather than assuming logging is broken.

## 12. Practical exercises on Ubuntu Server

Use a test VM and a non-production account. These exercises create and modify local lab resources.

### Exercise 1 — Explore the filesystem

~~~bash
pwd
ls -la /
ls -la /etc
ls -la /var/log
cd ~
~~~

Identify the purpose of /etc, /home, /var/log and /tmp.

### Exercise 2 — Create a user and group

~~~bash
sudo adduser labuser
sudo groupadd labteam
sudo usermod -aG labteam labuser
id labuser
groups labuser
~~~

Confirm that the user exists and belongs to the new group.

### Exercise 3 — Practice permissions

~~~bash
mkdir -p ~/linux-lab
cd ~/linux-lab
touch notes.txt
chmod 640 notes.txt
ls -l notes.txt
sudo chown labuser:labteam notes.txt
ls -l notes.txt
~~~

Explain what 640 means and identify the owner and group.

### Exercise 4 — Inspect a service and its logs

~~~bash
systemctl status ssh
sudo journalctl -u ssh -n 20
~~~

If SSH is not installed, install OpenSSH Server first. Observe the service state and recent log entries.

### Exercise 5 — Inspect disks and memory

~~~bash
lsblk -f
df -h
free -h
~~~

Identify the root filesystem, available disk space and memory usage. Do not format or repartition a disk as part of this exercise.

### Exercise 6 — Schedule a harmless task

Open the crontab:

~~~bash
crontab -e
~~~

Add this line to write the date to a file every minute:

~~~text
* * * * * date >> /home/YOUR_USERNAME/cron-test.log
~~~

Replace YOUR_USERNAME with your actual Linux username. Wait a few minutes and inspect the file:

~~~bash
cat ~/cron-test.log
~~~

Remove the test entry after confirming it works.

## 13. Commands to keep handy

| Task | Command |
|---|---|
| Current user | whoami |
| User and group IDs | id |
| Current directory | pwd |
| List files including hidden | ls -la |
| Move to home directory | cd ~ |
| Create directory | mkdir -p name |
| Copy / move files | cp source destination / mv source destination |
| Read a file | cat file / less file |
| Search text | grep |
| Create user | sudo adduser username |
| Create group | sudo groupadd groupname |
| Add user to group | sudo usermod -aG groupname username |
| List groups | groups username |
| Change permissions | chmod |
| Change ownership | chown |
| View ACL | getfacl file |
| List processes | ps aux |
| Live process view | top |
| Memory usage | free -h |
| Service status | systemctl status service |
| Service logs | journalctl -u service |
| Failed services | systemctl --failed |
| IP addresses | ip address |
| Routing table | ip route |
| Disk devices | lsblk -f |
| Filesystem space | df -h |
| Directory space | du -sh directory |
| Install package | sudo apt install package |
| Update package lists | sudo apt update |
| SSH connection | ssh user@server-ip |
| Edit cron schedule | crontab -e |
| View cron schedule | crontab -l |

## 14. Questions to revise

1. **What is the Linux root directory?**  
   The top-level directory of the Linux filesystem hierarchy, represented by /.

2. **What is the difference between /root and /?**  
   / is the top of the filesystem tree. /root is the home directory of the root user.

3. **What is the purpose of /etc?**  
   It stores system-wide configuration files.

4. **What is the purpose of /var/log?**  
   It commonly stores system and application log files.

5. **What is the difference between a user and a group?**  
   A user represents an account identity. A group collects users so permissions can be assigned to multiple accounts.

6. **What does sudo do?**  
   It allows an authorized user to execute a command with elevated privileges.

7. **What does chmod 755 file mean?**  
   The owner gets read, write and execute permissions. Group and others get read and execute permissions.

8. **What does chown do?**  
   It changes the owner and, optionally, the group of a file or directory.

9. **What is a process ID?**  
   A PID is the numeric identifier assigned to a process.

10. **What is systemd?**  
    It is the service manager and init system used by Ubuntu Server.

11. **What is the difference between start and enable in systemctl?**  
    start runs a service now. enable configures it to start automatically during boot.

12. **What is journalctl used for?**  
    It reads and filters logs stored in the systemd journal.

13. **What is SSH?**  
    SSH is an encrypted protocol used for remote login and command execution.

14. **What is a mount point?**  
    It is a directory where a filesystem is attached and made accessible.

15. **What is /etc/fstab?**  
    It contains filesystem mount configuration, including filesystems intended to mount automatically.

16. **What is cron?**  
    It is a time-based job scheduler used to run commands or scripts automatically.

17. **How would you investigate a service that is not starting?**  
    Check its status with systemctl, inspect its journal with journalctl, review the configuration and dependencies, correct the cause, then start it and verify the result.

## 15. Revision checklist

- [ ] Navigate the Linux filesystem and explain common directories.
- [ ] Use absolute and relative paths.
- [ ] Create, copy, move and remove files and directories.
- [ ] Identify users, groups and account information.
- [ ] Create a user and group and manage group membership.
- [ ] Explain sudo and least privilege.
- [ ] Read and change file permissions.
- [ ] Change file ownership and inspect ACLs.
- [ ] List and inspect processes.
- [ ] Start, stop, enable and inspect systemd services.
- [ ] Read service logs with journalctl.
- [ ] Explain SSH and connect to a server.
- [ ] Inspect disks, filesystems and mount points.
- [ ] Explain /etc/fstab and why it must be checked carefully.
- [ ] Install packages with APT.
- [ ] Create and inspect a cron job.
- [ ] Troubleshoot basic service, disk and network issues.

## Official documentation

- [Ubuntu Server documentation](https://documentation.ubuntu.com/server/)
- [Ubuntu community help — users and groups](https://help.ubuntu.com/community/AddUsersHowto)
- [systemd documentation](https://www.freedesktop.org/wiki/Software/systemd/)
- [OpenSSH manual](https://man.openbsd.org/ssh)
- [GNU Coreutils manual](https://www.gnu.org/software/coreutils/manual/)
