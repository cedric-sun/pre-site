Same-Origin Policy
================
Great security concerns appears if JavaScript supplied by `a.com` (embeded in HTML `<script>...</script>` or referred by `<script src="..."></script>`) can make `XMLHttpRequest` to `https?://b.com/.*`, since the HTTP cookie mechanism requires the browser to attach all cookies applicable for `b.com` with that `XMLHttpRequest`, even it's fired by scripts from `a.com`.

Thus modern browsers are all built with an security policy known as the "Same-Origin Policy", prohibiting scripts supplied by origin A from sending `XMLHttpRequest` to origin B. An "origin" is defined as a combination of (scheme, host, port).

Relaxing the SOP
====================
It then become a problem for large site with multiple sub-domains, e.g. scripts supplied by `news.company.com` may want to make `XMLHttpRequest` to `assets.company.com` to load resources. Historically people use the `window.document.domain` property trick to bypass SOP, but modern way to relaxing the SOP is CORS.

Cross-Origin Resource Sharing (CORS)
=============================


JSONP
=============


WebSocket
================


Fetch API
=================
