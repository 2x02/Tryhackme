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

Then I bruteforce for directory using dirsearch.py. But found nothing with that search.

--> sudo python3 /opt/dirsearch/-dirsearch.py -u http://10.10.28.239/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e php,html,apsx -x 404,403


