# -RootMe-TryHackMe-Notes
 Comprehensive walkthrough and practical notes for the RootMe machine on TryHackMe, focusing on enumeration, exploitation, and privilege escalation.
## 🚀 Step 1: Connect to TryHackMe VPN  
Before starting, **always connect to the TryHackMe VPN** to access the target network securely.  
Without VPN, your machine won’t be able to reach the target IP.

---

## 🔍 Step 2: Nmap Scan - Discover Open Ports & Services

Run a detailed and fast Nmap scan with:


nmap -T5 -O -sV 10.10.145.133 -oN nmap.results

What do these options mean?

    -T5: Maximum speed, aggressive timing for faster scan

    -O: Operating system detection

    -sV: Service version detection — crucial for finding version-specific vulnerabilities

    -oN: Save output in a readable format for later review

Optional output formats:

You can also save scans in different formats to use with other tools or scripts:
Format	Option	Purpose
Grepable	-oG nmap.grep	Easy to grep through results
XML	-oX nmap.xml	Structured output for tools
Script Kiddie	-oS nmap.script	Funny but rarely used format
All formats	-oA allformats	Saves all above formats

    💡 Pro tip: Always use -sV! Vulnerabilities are often specific to service versions.

🔐 Step 3: Directory Brute-Forcing for Hidden Paths

Use tools like Dirbuster or Gobuster to discover hidden directories or files on the webserver.

gobuster dir -u http://10.10.145.133 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

Why?

Many websites hide admin panels or important files under non-obvious paths — brute forcing helps find these.
📂 Step 4: Analyze Website & Check robots.txt

Visit the site in your browser:

http://10.10.145.133

Check /robots.txt:

http://10.10.145.133/robots.txt

Developers sometimes list disallowed directories here, which may contain sensitive files or hints.
🛠️ Step 5: Found /panel/ Directory - Upload Reverse Shell
Why PHP reverse shell?

    PHP files are executed by the web server (Apache).

    Executables like .exe won’t run in a web environment.

    Other scripting files like JavaScript often get blocked or are client-side only.

Important: Bypass PHP Upload Restrictions

Often, .php uploads are blocked (blacklisted). You can try:

    Changing extension to .php3, .php4, .php5 (older PHP versions accepted by server).

    This bypasses blacklist filtering since it’s based on extension blocking, not whitelist.

🐚 Step 6: Prepare PHP Reverse Shell

    Download PHP reverse shell from PentestMonkey

    Change the IP and port to your TryHackMe VPN IP and listening port (4444 recommended).

    Upload your modified shell file to /panel/.

📡 Step 7: Start Netcat Listener

Open your terminal and run:

nc -lvnp 4444

Explanation:

    n: No DNS lookup (faster output)

    l: Listen mode (wait for incoming connection)

    v: Verbose mode (show details)

    p 4444: Listen on port 4444

💥 Step 8: Trigger the Reverse Shell

Visit the URL where your shell is uploaded:

http://10.10.145.133/panel/<your_shell_filename>

If everything works, you’ll get a shell prompt on your Netcat listener — you now have command execution on the server!
🔎 Step 9: Manual Enumeration on Target Server

Explore the file system to find:

    Credentials

    Flags

    Useful files for privilege escalation

Start by checking /var/www for the first flag.
🔒 Step 10: Privilege Escalation Using SUID Binaries
What is SUID?

SUID (Set User ID) is a special permission allowing executables to run with the file owner’s privileges (often root).

Attackers scan for SUID binaries to escalate privileges.
🔍 Step 11: Find SUID Files

Run:

find / -perm -4000 -type f 2>/dev/null

    Searches entire system for files with SUID bit set.

    Suppresses permission denied errors.

⚠️ Step 12: Example SUID Exploit - Python

If /usr/bin/python has SUID root permissions, run:

python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

This spawns a root shell because Python inherits root permissions.
✅ Summary & Key Takeaways

    Always scan with nmap -sV to find versions — essential for exploits.

    Use directory brute forcing to uncover hidden paths like /panel/.

    PHP reverse shell is preferred for web server exploitation; try extension bypasses.

    Netcat listener must be running before triggering the shell.

    Manually explore server for flags and sensitive files.

    SUID binaries are common privilege escalation vectors.

    Exploit SUID Python to gain root shell quickly if available.

📚 Resources

    PentestMonkey PHP Reverse Shell

    TryHackMe

    Nmap Official Documentation

    Gobuster GitHub

Happy Hacking! 🕵️‍♂️🔐
