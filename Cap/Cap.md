
# Cap - Retired Machine

**About Cap**\
Cap is an easy difficulty Linux machine running an HTTP server that performs administrative functions including performing network captures. Improper controls result in Insecure Direct Object Reference (IDOR) giving access to another user's capture. The capture contains plaintext credentials and can be used to gain foothold. A Linux capability is then leveraged to escalate to root.

## Step 1 - Nmap Scan
Just performed basic nmap scan:\
`nmap -sC -SV <Target-IP>`\
where:\
`-sC` - Scan with default NSE scripts. Considered useful for discovery and safe.\
`-sV` - Attempts to determine the version of the service running on port.\

The Output it produced is in the file - [Nmap output file](https://github.com/Nithin-vs/HTB-Labs-Docs/blob/main/Cap/Nmap.txt)

## Step 2 - Opening in Browser

As the Nmap shows port 80 HTTP is open, just opening hosts file in kali, `sudo nano /etc/hosts` and adding the target IP to the last.

Basic Surfing through the Dashboard, found there is a sub-divison going to Security Snapshot when opening it, URL shows the plaintext to the directories. 

The URL is like `http://<Target-IP>/data/8`, just using the **IDOR**, changed the 8 to 0. Found there is more data.


![Security dashboard](https://github.com/Nithin-vs/HTB-Labs-Docs/blob/main/Cap/Security%20Dashboard.png)

Downloaded the **PCAP** file and opened it in the Wireshark and used FTP filter.


![Pcap file](https://github.com/Nithin-vs/HTB-Labs-Docs/blob/main/Cap/Pcap%20File.png)

Found the password for the user Nathan. It is also used for SSH login.

## Step 3 - SSH login
Using the Creds what I get in wireshark, successfully logged into the user. Where I found the user flag.
 
It was easy till here.

Finding the Root flag is not as easy as finding user flag.

## Step 4 - Root flag
By creating a `exploit.py` file using:
```
Import os
os.setuid(0)
os.system("/bin/bash")
```
and executing it gives me root shell.

By navigating to the `/root` directory found the Root flag.

![Cap has been pawned](https://github.com/Nithin-vs/HTB-Labs-Docs/blob/main/Cap/Achieved.png)
