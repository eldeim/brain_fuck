# Databases & Queries

## Command Line

```bash
mysql -u root -h docker.hackthebox.eu -P 3306 -p 

Enter password: 
...SNIP...

## or
mysql -u root -h 94.237.49.101 -P 44324 -p
```

## Basics Query

<pre class="language-sql"><code class="lang-sql"><strong>USE dbtest;
</strong><strong>SELECT * FROM table_name;
</strong>SELECT column1, column2 FROM table_name;
</code></pre>

### LIKE Clause

```shell-session
mysql> SELECT * FROM logins WHERE username LIKE 'admin%';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | administrator | adm1n_p@ss | 2020-07-02 15:19:02 |
+----+---------------+------------+---------------------+

mysql> SELECT * FROM logins WHERE username like '___';

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  3 | tom      | tom123!  | 2020-07-02 15:18:56 |
+----+----------+----------+---------------------+

select * from employees where first_name like 'Bar%' and hire_date like '1990-01-01';
```

### Operators in queries

```shell-session
mysql> SELECT * FROM logins WHERE username != 'john';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
3 rows in set (0.00 sec)
```

The next query selects users who have their `id` greater than `1` AND `username` NOT equal to `john`:

&#x20; SQL Operators

```shell-session
mysql> SELECT * FROM logins WHERE username != 'john' AND id > 1;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```

* Division (`/`), Multiplication (`*`), and Modulus (`%`)
* Addition (`+`) and subtraction (`-`)
* Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
* NOT (`!`)
* AND (`&&`)
* OR (`||`)

```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
```
