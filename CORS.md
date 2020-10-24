Same-Origin Policy
================
An FQDN is said to supplied a JavaScript snippet if the snippet is eventually downloaded due to its being embeded in `<script>...</script>` or referred by `<script src="..."></script>` inside the HTML text returned from that FQDN.

e.g. `http://example.com/index.html` contains `<script src="https://www.google-analytics.com/analytics.js"></script>`, then `example.com` is said to supply `analytics.js`, **not `www.google-analytics.com`**.

Great security concerns appears if JavaScript supplied by `a.com`  can make `XMLHttpRequest` to `https?://b.com/.*`, since the HTTP cookie mechanism requires the browser to attach all cookies applicable for `b.com` with that `XMLHttpRequest`, even it's fired by scripts from `a.com`.

Thus modern browsers are all built with an security policy known as the "Same-Origin Policy", prohibiting scripts supplied by origin A from sending `XMLHttpRequest` to origin B. An "origin" is defined as a combination of (scheme, host, port).

Note that SOP is a security policy enforced by the user browser. Every time scripts make a XHR, the browser check whether the origin triplet `(target-scheme,target-host, target-port)` of the target of that XHR is the same as the origin triplet `(script-scheme,window.document.domain,script-port)`.

Relaxing the SOP
====================
It then become a problem for large site with multiple sub-domains, e.g. scripts supplied by `news.company.com` may want to make `XMLHttpRequest` to `assets.company.com` to load resources. Historically people use the `window.document.domain` property trick to bypass SOP. Scripts are allowed to strip prefixes of that property to make the SOP check pass.

Modern way to relaxing the SOP is CORS.

Cross-Origin Resource Sharing (CORS)
=============================


JSONP
=============


WebSocket
================


Fetch API
=================
