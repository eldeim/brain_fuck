# Deobfuscation

## Beautify

For example, if we were using Firefox, we can open the browser debugger with \[ `CTRL+SHIFT+Z` ], and then click on our script `secret.js`. This will show the script in its original formatting, but we can click on the '`{ }`' button at the bottom, which will `Pretty Print` the script into its proper JavaScript formatting:

<figure><img src="https://academy.hackthebox.com/storage/modules/41/js_deobf_pretty_print.jpg" alt=""><figcaption></figcaption></figure>

Furthermore, we can utilize many online tools or code editor plugins, like [Prettier](https://prettier.io/playground/) or [Beautifier](https://beautifier.io/). Let us copy the `secret.js` script:

Code: javascript

```javascript
eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('g 4(){0 5="6{7!}";0 1=8 a();0 2="/9.c";1.d("e",2,f);1.b(3)}', 17, 17, 'var|xhr|url|null|generateSerial|flag|HTB|flag|new|serial|XMLHttpRequest|send|php|open|POST|true|function'.split('|'), 0, {}))
```

We can see that both websites do a good job in formatting the code:

&#x20;     \


<figure><img src="https://academy.hackthebox.com/storage/modules/41/js_deobf_beautifier_1.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://academy.hackthebox.com/storage/modules/41/js_deobf_prettier_1.jpg" alt=""><figcaption></figcaption></figure>

