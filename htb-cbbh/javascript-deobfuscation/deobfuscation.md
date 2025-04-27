# Deobfuscation

## Beautify

For example, if we were using Firefox, we can open the browser debugger with \[ `CTRL+SHIFT+Z` ], and then click on our script `secret.js`. This will show the script in its original formatting, but we can click on the '`{ }`' button at the bottom, which will `Pretty Print` the script into its proper JavaScript formatting:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Furthermore, we can utilize many online tools or code editor plugins, like [Prettier](https://prettier.io/playground/) or [Beautifier](https://beautifier.io/). Let us copy the `secret.js` script:

Code: javascript

```javascript
eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('g 4(){0 5="6{7!}";0 1=8 a();0 2="/9.c";1.d("e",2,f);1.b(3)}', 17, 17, 'var|xhr|url|null|generateSerial|flag|HTB|flag|new|serial|XMLHttpRequest|send|php|open|POST|true|function'.split('|'), 0, {}))
```

We can see that both websites do a good job in formatting the code:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

We can find many good online tools to deobfuscate JavaScript code and turn it into something we can understand. One good tool is [UnPacker](https://matthewfl.com/unPacker.html). Let's try copying our above-obfuscated code and run it in UnPacker by clicking the `UnPack` button.

> Tip: Ensure you do not leave any empty lines before the script, as it may affect the deobfuscation process and give inaccurate results.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
