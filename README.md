 🛠️ Rootme Machine Notes - TryHackMe

> These are my detailed notes from solving the **Rootme** machine on TryHackMe.  
> This work was done in a **virtual lab environment for learning and practice only**.  
> No unauthorized systems were accessed.

---

## 🌐 Step 1: Connect to TryHackMe VPN Network

To interact with the TryHackMe machines, first connect to their VPN network.  
This connection allows your computer to reach the isolated lab environment where the target machine (`10.10.145.133`) is hosted.

----

## 🔍 Step 2: Nmap Scan to Discover Open Ports and Services

Run the following command to scan the target:


nmap -T5 -O -sV 10.10.145.133 -oN nmap.results

What does each option mean?

    -T5 — Use the fastest scanning speed (aggressive timing). This speeds up the scan but may increase the chance of detection.

    -O — Enable operating system detection, so Nmap tries to guess what OS the target is running.

    -sV — Service version detection. This tells you exactly what service is running on each open port and its version number.

    -oN — Save the scan output in a normal human-readable format to a file named nmap.results.

Why use -sV?

Because vulnerabilities depend on specific versions of software. Knowing the exact version helps find applicable exploits.
📂 Optional Nmap Output Formats

Nmap supports different output formats for different purposes:

    -oG — Grepable format. Easier to search with tools like grep.

    -oX — XML format. Used for automated tools and parsing.

    -oS — Script Kiddie format. Mostly for fun, not used seriously.

    -oA — Saves all output types at once.

🗂️ Step 3: Directory Bruteforcing to Find Hidden Paths

Web applications often have hidden directories that are not linked anywhere but contain important resources.

Use tools like Dirbuster or Gobuster to brute force directories:

gobuster dir -u http://10.10.145.133 -w /usr/share/wordlists/dirb/common.txt

Look for directories that might give you entry points. Here, /panel/ was discovered.
🖥️ Step 4: Analyze Website and Source Code

Visit the discovered /panel/ URL. Look at:

    The page source for hidden comments or credentials.

    The robots.txt file (e.g., http://10.10.145.133/robots.txt), which might reveal directories the admin doesn’t want crawled but could be interesting to us.

🐚 Step 5: Upload PHP Reverse Shell to Gain Server Access

To take control of the server, upload a reverse shell script. A reverse shell connects back from the server to your machine, letting you run commands remotely.
Why PHP reverse shell?

    The server runs an Apache web server that understands and executes .php files.

    Uploading .exe or other binaries won't work because browsers and servers won't run them.

    PHP scripts are executed by the server, so a PHP shell can run commands for you.

🚧 Step 6: Bypass File Upload Restrictions

Most servers block uploading .php files for security.

Try to:

    Rename your file to .php3, .php4, or .php5 — older PHP extensions that may bypass filters.

    Servers might blacklist .php extensions but allow others, rather than strictly whitelisting allowed file types.

This trick can let your reverse shell file upload successfully.
🔌 Step 7: Prepare PHP Reverse Shell Script

Get the PHP reverse shell script from PentestMonkey.

Edit the script:

    Replace the IP (LHOST) with your TryHackMe VPN IP.

    Replace the port (LPORT) with your listening port (e.g., 4444).

📡 Step 8: Start Netcat Listener

Before triggering the shell, listen on your machine for the incoming connection:

nc -lvnp 4444

What do the flags mean?

    l — Listen mode (wait for connections).

    v — Verbose output (show connection details).

    n — No DNS resolution (faster output).

    p 4444 — Listen on port 4444.

0.0.0.0 means listen on all network interfaces (VPN, localhost, etc.).
🎉 Step 9: Execute the Reverse Shell

Browse to the uploaded reverse shell URL:

http://10.10.145.133/panel/shell.php3

When accessed, the PHP script connects back to your machine, giving you shell access.
🔎 Step 10: Manual Enumeration on the Server

Once you have shell access, explore the system manually:

    Look for sensitive files containing credentials or flags.

    Check common directories like /var/www for web content or hidden files.

🔼 Step 11: Find SUID Binaries for Privilege Escalation

SUID (Set User ID) allows executables to run with the file owner's permissions, often root.

Attackers use SUID files to escalate privileges.

Find all SUID files:

find / -perm -4000 -type f 2>/dev/null

    find / — Search entire system.

    -perm -4000 — Files with SUID bit set.

    -type f — Only files.

    2>/dev/null — Hide errors.

🐍 Step 12: Exploit SUID Python to Get Root Shell

If /usr/bin/python has SUID bit and is root-owned, run:

python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

Explanation:

    python -c — Run Python code passed as a string.

    import os — Import OS module.

    os.execl("/bin/sh", "sh", "-p") — Execute shell with preserved privileges.

Because Python runs with root privileges (due to SUID), this launches a root shell.
⚠️ Important Notes

    Always use -sV with Nmap to find service versions for better exploit targeting.

    File upload bypasses depend on the server’s blocking strategy (blacklist vs whitelist).

    SUID binaries are a critical part of privilege escalation.

    This work was done in a controlled lab for learning.

🏁 Summary Commands

nmap -T5 -O -sV 10.10.145.133 -oN nmap.results
gobuster dir -u http://10.10.145.133 -w /usr/share/wordlists/dirb/common.txt
find / -perm -4000 -type f 2>/dev/null
nc -lvnp 4444
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

 References

    TryHackMe Rootme Room

    PentestMonkey PHP Reverse Shell
