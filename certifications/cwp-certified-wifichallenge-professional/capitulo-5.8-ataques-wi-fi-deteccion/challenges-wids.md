# Challenges WIDS

## Challenge 27

* ¿Cuál es la MAC del primer atacante detectado en Nzyme?

Al abrir la web, nos hace un redirect al localhost y vemos el panel de nzyme

<figure><img src="../../../.gitbook/assets/image (1196).png" alt=""><figcaption></figcaption></figure>

> `admin : admin`

<figure><img src="../../../.gitbook/assets/image (1197).png" alt=""><figcaption></figcaption></figure>

Dentro, vemos un panel de alertas -->

<figure><img src="../../../.gitbook/assets/image (1198).png" alt=""><figcaption></figcaption></figure>

En uno de estos, vemos un ataque con un BSSID no reconocido, donde explica hacia que red ha sido y que MAC buscando por (`UNEXPECTED_BSSID_PROBERESP`)

<figure><img src="../../../.gitbook/assets/image (1200).png" alt=""><figcaption></figcaption></figure>
