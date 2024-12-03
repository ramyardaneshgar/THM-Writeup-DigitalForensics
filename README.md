# THM-Writeup-DigitalForensics
Writeup for TryHackMe "Disgruntled" Lab - security tools, commands used, and key lessons learned.

https://tryhackme.com/r/room/disgruntled

# Background
The "Disgrunteld" was an lab in digital forensics, focusing on identifying and investigating malicious activity on a compromised system. An employee was suspected of deploying a "logic bomb" or other malicious actions. My task was to analyze system logs, identify unauthorized changes, and track any suspicious activity to understand the scope of the compromise.


## **Steps I Took**

### **Task 1: System Setup**
1. I connected to the virtual machine provided in the lab using SSH with the credentials:
   ```bash
   ssh root@<IP address>
   ```
2. After gaining access, I ensured that I understood the structure and location of system logs, primarily focusing on `/var/log/auth.log` for authentication and sudo-related activity.

---

### **Task 2: Investigating Suspicious Commands**

#### **Identifying Privileged Commands**
- I examined the authentication logs for commands executed with elevated privileges:
  ```bash
  cat /var/log/auth.log | grep install
  ```
- This revealed the command used to install a package and the working directory (`PWD`) where the command was executed.

#### **Tracking User Creation**
- Using a similar approach, I filtered the logs for user creation commands:
  ```bash
  cat /var/log/auth.log | grep useradd
  ```
- This helped me identify a new user added to the system.

#### **Checking Sudo Privileges**
- I searched for edits to the `/etc/sudoers` file by filtering for `visudo` commands:
  ```bash
  cat /var/log/auth.log | grep visudo
  ```
- This allowed me to pinpoint when the file was updated to grant sudo privileges to the newly created user.

#### **Locating Edited Scripts**
- I checked for files opened with the `vi` editor, which revealed a suspicious script file:
  ```bash
  cat /var/log/auth.log | grep vi
  ```

---

### **Task 3: Tracking Malicious Files**

#### **Locating Created Files**
- I navigated to the home directory of the newly created user:
  ```bash
  cd /home/it-admin
  ls -la
  ```
- Hidden files and the `.bash_history` file provided valuable clues:
  ```bash
  cat .bash_history
  ```

#### **Tracing Renamed Files**
- By examining the user’s history file, I discovered that the suspicious script was renamed and moved to another directory. This required checking `/etc/crontab` for its new location:
  ```bash
  cat /etc/crontab
  ```

#### **Inspecting File Metadata**
- Using the `ls -lt` command, I retrieved the last modified timestamp for the file:
  ```bash
  ls -lt /path/to/file
  ```

#### **Understanding the Script’s Functionality**
- I read the content of the script to identify its purpose and the name of the file it generates:
  ```bash
  cat <script>
  ```

---

### **Task 4: Understanding Scheduled Execution**

#### **Decoding Cron Jobs**
- The script was set to execute via a cron job. I analyzed the cron schedule from `/etc/crontab`:
  ```bash
  cat /etc/crontab
  ```
- To convert the schedule into a human-readable format, I used [crontab.guru](https://crontab.guru).

---

## **Lessons Learned**

### **1. The Importance of Log Files**
System logs like `/var/log/auth.log` are critical for understanding a threat actor’s actions and timelines.

### **2. Tracing Insider Threats**
Insider threats often leave behind traces in command histories and edited files. Monitoring user activity is essential for detecting and preventing such threats.

### **3. Understanding Persistence Mechanisms**
Cron jobs and other scheduled tasks are common ways attackers ensure persistence. Familiarity with tools like `crontab` is essential in incident response.

### **4. Systematic Investigation Approach**
By starting with privileged commands and methodically tracing subsequent actions, I uncovered the full scope of the attacker’s activities.

---

## **Summary**
Through log analysis, file inspections, and cron job decoding, I identified the malicious actions of the disgruntled employee. They had created a "logic bomb" script designed to delete files after a period of inactivity, highlighting the risks of insider threats and the importance of effective monitoring and forensic analysis in cybersecurity.
```
