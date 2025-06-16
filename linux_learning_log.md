My Comprehensive Linux Learning Log
Date: 2025-06-16
This document serves as a detailed log of my Linux learning journey, focusing on Git/GitHub integration, firewall management (UFW vs. Firewalld), web server configuration (Apache), and essential Linux commands. All work is conducted within an Ubuntu WSL (Windows Subsystem for Linux) environment.

1. Git & GitHub Workflow for Project Management
1.1 Resolving GitHub Push Issues
Initially, I faced issues where local Git changes weren't appearing on GitHub. This was primarily due to:

Browser caching (requiring a refresh).

Previous git push commands not fully completing.

Authentication failures, as GitHub now requires a Personal Access Token (PAT) instead of a password for HTTPS pushes.

Solution:

Generate a Personal Access Token (PAT) on GitHub:

Go to GitHub's Token Settings.

Generate a new token (Fine-grained is preferred).

Set appropriate permissions (e.g., repo scope or Contents: Read and write for fine-grained).

Crucially, copy the token immediately upon generation as it cannot be viewed again.

Use the PAT as the password when git push prompts for credentials.

1.2 Remembering Git Credentials (PAT)
To avoid re-entering the PAT for every push, I configured Git to store credentials:

git config --global credential.helper store

This stores the PAT in plain text in ~/.git-credentials, which Git then uses automatically for subsequent pushes. While convenient, it's less secure than other methods like cache which stores it temporarily.

1.3 Standard Git Push Workflow
My regular workflow for pushing changes to GitHub is:

Navigate to the repository directory:

cd ~/my_linux_practice

Stage all changes (new and modified files):

git add .

Commit changes with a descriptive message:

git commit -m "Descriptive message about changes"

Push committed changes to GitHub:

git push

(If remote origin is not set or needs updating, use git remote set-url origin https://github.com/ramzanchowdhary/my_linux_practice.git before pushing, and git branch -M main if changing branch name.)

2. Firewall Management: UFW vs. Firewalld in WSL
I explored two primary Linux firewall management tools: Firewalld and UFW (Uncomplicated Firewall). Due to the nature of WSL, UFW proved to be the more practical choice.

2.1 Firewalld Challenges in WSL
Installation: sudo apt install firewalld

Status Check: sudo systemctl status firewalld

Problem: Firewalld repeatedly failed to start (inactive (dead)) with errors like 'python-nftables' failed and Error: Could not process rule: No such file or directory.

Reason: Firewalld's reliance on nftables (a modern firewall backend) and deeper kernel interactions is often incompatible with the virtualized networking stack of WSL.

Conclusion: Firewalld is not recommended for WSL environments unless advanced kernel adjustments are made.

2.2 Transitioning to UFW
I successfully disabled Firewalld and enabled UFW:

Stop and Disable Firewalld:

sudo systemctl stop firewalld
sudo systemctl disable firewalld

Install UFW (if not already installed):

sudo apt install ufw

Enable UFW:

sudo ufw enable

Set Default Policies (recommended for security):

sudo ufw default deny incoming
sudo ufw default allow outgoing

2.3 Key UFW Commands and Service Management
UFW manages firewall rules by explicitly allowing or denying traffic based on ports, protocols, or predefined application profiles.

Check UFW status:

Basic status: sudo ufw status

Detailed status (including policies, logging): sudo ufw status verbose

Service status via systemd: sudo systemctl status ufw

Allowing Services/Ports:

SSH (port 22): sudo ufw allow ssh or sudo ufw allow 22/tcp

HTTP (port 80): sudo ufw allow http or sudo ufw allow 80/tcp

HTTPS (port 443): sudo ufw allow https or sudo ufw allow 443/tcp

Apache (combines HTTP/HTTPS, if profile available): sudo ufw allow "Apache Full"

CUPS (port 631): sudo ufw allow 631/tcp (used direct port as profile not initially found)

Samba (ports 137-139, 445): sudo ufw allow 137/udp, sudo ufw allow 138/udp, sudo ufw allow 139/tcp, sudo ufw allow 445/tcp (used direct ports as profile not initially found)

Custom ports (e.g., 8080, 20201): sudo ufw allow 8080/tcp, sudo ufw allow 20201/tcp, sudo ufw allow 20201 (for both TCP/UDP)

Removing Allowed Rules:

By service name: sudo ufw delete allow ssh

By port/protocol: sudo ufw delete allow 8080/tcp

Listing Available UFW Application Profiles:

sudo ufw app list

(Profiles like Apache, OpenSSH, CUPS, Samba appear here if their respective packages are installed and register a profile.)

Reloading UFW:
UFW applies changes immediately, so reload is often not strictly necessary but can be used:

sudo ufw reload

A "soft restart" can be done with: sudo ufw disable && sudo ufw enable

Resetting UFW:

sudo ufw reset

Caution: This command completely deletes all rules and disables UFW. Rules need to be re-added manually after a reset.

2.4 Blocking IP Addresses in UFW
To block all incoming traffic from a specific IP address:

sudo ufw deny from 192.168.1.50

To block traffic from an IP to a specific port:

sudo ufw deny from 192.168.1.50 to any port 80

To remove a deny rule:

sudo ufw delete deny from 192.168.1.50

2.5 Blocking ICMP (Ping) in UFW
UFW doesn't have a direct command for this. It involves editing a core rule file.

Open /etc/ufw/before.rules:

sudo nano /etc/ufw/before.rules

Add the following line (before the COMMIT line in the *filter section) to drop incoming echo-requests:

-A ufw-before-input -p icmp --icmp-type echo-request -j DROP

Save, exit, and reload UFW:

sudo ufw reload

Note: Completely blocking ICMP is generally not recommended as it can affect network diagnostics and essential functions like Path MTU Discovery.

3. Web Server Configuration (Apache2)
I configured the Apache web server (apache2 package on Ubuntu) and resolved a common port binding issue in WSL.

Installation:

sudo apt install apache2

Enabling Common Modules:

sudo a2enmod rewrite ssl headers proxy proxy_http

Troubleshooting "Address already in use" on Port 80:

Apache failed to start because wslrelay.exe (a Windows process related to WSL networking) was already using port 80.

Solution: Change Apache's listening port to an alternative, like 8080.

Edit /etc/apache2/ports.conf: Change Listen 80 to Listen 8080.

Edit /etc/apache2/sites-enabled/000-default.conf: Change <VirtualHost *:80> to <VirtualHost *:8080>.

Allow the new port in UFW: sudo ufw allow 8080/tcp

Restart Apache: sudo systemctl restart apache2

Verifying Apache is Running:

sudo systemctl status apache2

(Look for Active: active (running))

Accessing Apache from Windows:
After changing the port, access the web server from a Windows browser using the WSL IP and the new port: http://<WSL_IP_ADDRESS>:8080/

4. Fundamental Linux Commands & Concepts
ls command: Lists directory contents.

ls: List current directory contents.

ls -l: Long listing format (permissions, owner, size, date).

ls -a: List all files, including hidden ones (starting with .).

ls -lh: Long listing with human-readable file sizes.

ls -R: Recursive listing (contents of subdirectories).

find command: Searches for files/directories.

find ~ -name "my_file.txt": Find my_file.txt in home directory.

find ~ -type f -newermt "YYYY-MM-DD": Find files modified since a specific date.

find ~ -type f \( -name "*.sh" -o -name "*.txt" -o -name "*.md" \): Find all shell scripts, text files, or Markdown files.

locate command: Fast file search (requires mlocate package and sudo updatedb).

Installation: sudo apt install mlocate

Update database: sudo updatedb

Search: locate myfile.txt

mv command: Move or rename files/directories.

mv /path/to/source/file /path/to/destination/

Example: mv ~/Downloads/notes.txt ~/my_linux_practice/

cd command: Change directory.

cd: Go to home directory.

cd ~: Go to home directory.

cd ..: Go up one directory.

pwd command: Print Working Directory (shows current location).

file command: Determines file type by inspecting content.

file my_document.pdf

Viewing file content (cat, less, head, tail, nano):

cat filename: Prints entire content (for short files).

less filename: Opens in a pager (for long files, allows scrolling).

head filename: Shows first 10 lines.

tail filename: Shows last 10 lines.

nano filename: Opens in a terminal text editor.

5. Restarting WSL Ubuntu
Recommended method: Shut down the WSL subsystem from Windows:

wsl --shutdown

Then, open a new Ubuntu terminal.

sudo reboot inside WSL only restarts Linux processes, not the underlying WSL VM.

This log covers the key aspects of our extensive discussions and troubleshooting. It should be a valuable resource for your ongoing Linux learning!
