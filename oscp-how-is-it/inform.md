---
description: ~ 24 Horas para hacer el informe
---

# 📄 Inform

* **Recomendable haber hecho al menos 1 de prueba antes**
* **Consideraciones**
  * Mantener una estructura
  * Secciones exp, fix, severity y steps a reproducir ...
  * Poner links al exploit utilizado (Si se hacen modificaciones, exponerlas y porque se hicieron)
*   [**Capturas Flags (local.txt y proof.txt) (debe de aparecer:  )**](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide-Newly-Updated#screenshot-requirements)

    * id / whoami
    * hostname
    * ipconfig / ip a
    * cat / type (local.txt / proof.txt)

    <figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
*   [**Enviar en PDF dentro de un 7z con nombre especifico (desde kali)**](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide-Newly-Updated#section-3-submission-instructions)

    * Verificar con MD5

    ```
    ┌──(kali㉿kali)-[~]
    └─$ sudo md5sum OSCP-OS-XXXXX-Exam-Report.7z
    f7feecea01ac1eca9ee522906b087d5e OSCP-OS-XXXXX-Exam-Report.7z
    ```



    * Verificar genralmente

    ```
    ┌──(kali㉿kali)-[~]
    └─$ sudo 7z a OSCP-OS-XXXXX-Exam-Report.7z OSCP-OS-XXXXX-Exam-Report.pdf

    7-Zip 9.20 Copyright (c) 1999-2010 Igor Pavlov 2010-11-18 p7zip Version 9.20 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,2 CPUs)

    Scanning

    Updating archive OSCP-OS-XXXXX-Exam-Report.7z


    Everything is Ok
    ```



    * Enviar a [upload.offsec.com](https://upload.offsec.com/) con su MD5



## **Exam Report Template**

{% file src="../.gitbook/assets/OSCP-Exam-Report.odt" %}
