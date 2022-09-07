# Jurassic Park

Jurassic Park is a room from TryHackMe. [Here the link](https://tryhackme.com/room/jurassicpark).

The room's description says: 
"_This medium-hard task will require you to enumerate the web application, get credentials to the server and find 5 flags hidden around the file system..._"

The question we shoudl answer are the following:

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
    I used nmap for scan the most common port of our target. I used the OS Detection(-O) and Version Detection (-sV) flags in addition to the default scripts (-sC). Added a bit of verbosity (-v) and save the scan in a file called nmap_initial.
2. `rustscan -a 10.10.251.164 -r 1-65535 | tee scans/rustscan`
    I used rustscan for check if other ports, besides the well-know ones, are also open.

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

In one of the terminal I runned gobuster, for discover eventualy hidden folder.

`gobuster dir -u http://10.10.251.164 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee scans/gobuster`

