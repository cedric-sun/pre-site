Same-Origin Policy
================
An FQDN is said to supplied a JavaScript snippet if the snippet is eventually downloaded due to its being embeded in `<script>...</script>` or referred by `<script src="..."></script>` inside the HTML text returned from that FQDN.

e.g. `http://example.com/index.html` contains `<script src="https://www.google-analytics.com/analytics.js"></script>`, then `example.com` is said to supply `analytics.js`, **not `www.google-analytics.com`**. `analytics.js` can do anything with the DOM.

Great security concerns appears if JavaScript supplied by `a.com` can make `XMLHttpRequest` to `https?://b.com/.*`, since the HTTP cookie mechanism requires the browser to attach all cookies applicable for `b.com` with that `XMLHttpRequest`, even it's fired by scripts from `a.com`.

In real life, no information can be obtained about whether `b.com` allows such Cross-Origin request if browsers don't allow Cross-Origin request at all. So making request is allowed, but whether browsers allow the response to be read depends on the response.

Thus modern browsers are all built with an security policy known as the "Same-Origin Policy", prohibiting scripts supplied by origin A from reading response of `XMLHttpRequest` to origin B. An "origin" is defined as a combination of (scheme, host, port).

Note that SOP is a security policy enforced by the user browser. Every time scripts receive a XHR response, the browser check whether the origin triplet `(target-scheme,target-host, target-port)` of the target of that XHR is the same as the origin triplet `(script-scheme,window.document.domain,script-port)`.

https://security.stackexchange.com/a/191114

> The browser's insistence on sending cookies with every request to a given domain (which is pretty much a requirement for many web applications to work properly) is the main impetus behind the SOP and CORS restrictions.


Relaxing the SOP
====================
It then become a problem for large site with multiple sub-domains, e.g. scripts supplied by `news.company.com` may want to read response of XHR to `assets.company.com` to load resources. Historically people use the `window.document.domain` property trick to bypass SOP. Scripts are allowed to strip prefixes of that property to make the SOP check pass. Other means include `JSONP`.

Modern way to relax the SOP is CORS.

Cross-Origin Resource Sharing (CORS)
=============================
The whole mechanism of CORS is built upon 2 HTTP headers:

#### `Origin` request header
TODO

#### `Access-Control-Allow-Origin` response header
`Access-Control-Allow-Origin` appears in the HTTP response in response to the `Origin` header in the request.

Server that receive XHR request can set `Access-Control-Allow-Origin` header in the response to indicate to which FQDNs it's willing to share resources. Upon receive the XHR response, the browser will check if this header in the response contains the current origin.


Only one occurrence of this header is allowed in the response, and 

> Only a single origin can be specified. If the server supports clients from multiple origins, it must return the origin for the specific client making the request. (https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)


CSRF attack
==================
We see that SOP of the browser prevent malicious script from **reading the response** of a Cross-Origin request not allowed by the target origin. That also means that the browser allows malicious script to **send** Cross-Origin request. It would be dangerous if malicious scripts requests at vulnerable API at the target origin that only requires authenticated cookies.

This consideration impose a security requirement for Web API designer.

https://en.wikipedia.org/wiki/Cross-site_request_forgery#Prevention


WebSocket
================


Fetch API
=================
