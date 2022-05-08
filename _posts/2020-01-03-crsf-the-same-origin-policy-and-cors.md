---
layout: post
title:  "CSRF, the same-origin policy and CORS"
date:   2020-01-03
---

## CSRF

When a user is authenticated with a web service, an attacker can use cross-site request forgery (CSRF) to trick them into making a request to that service to perform a desired action.

If a service uses `GET` requests for state-changing operations, the attacker could present the user with a link that triggers an unwanted update. `POST` requests can be triggered with a form submission. Both can be generated without deliberate action from the user, for example by embedding an image tag that links to a protected resource on the target service, or by including some JavaScript that submits a form on a mouseover event. An attacker could include malicious links or code on their own site, or embed either elsewhere, particularly where there is a cross-site scripting vulnerability.

### Countermeasures

HTTP methods should be used as intended. Specifically, use `POST` not `GET` for state-changing operations. Assuming that this is the case, `GET` requests can be considered safe: even if a request is unintended, its response is just returned harmlessly to the user's browser.

`POST` requests can be protected by including a security token, verified by the server, in forms and AJAX requests, [as happens in Ruby on Rails][1].

`GET` requests for dynamic JavaScript resources can be vulnerable, as an attacker could include such a script in a `<script>` tag on their own page, and potentially extract user data after the script has executed (e.g. from a [global variable][2]). [Rails prevents embedding of JavaScript responses][3] by default for this reason.

## The same-origin policy

Whilst an attacker cannot use CSRF to steal data returned from a `GET` request, they could theoretically do so by executing a script that makes an AJAX request to a third-party service with the user's credentials and then forwards the response. Happily, browsers enforce the [same-origin policy][4] to protect against this.

The origin of a web page is generally the scheme, host and port taken together. Requests to a different origin than that of the requesting page are known as cross-origin requests. When a browser makes a cross-origin AJAX request, the same-origin policy ensures that it will raise an error and not share the response with the calling code. The policy is intended to isolate content from different sources within the browser, and prevents AJAX requests from being used to read a user's data from a third-party service.

'Simple' cross-origin requests (`GET`, `HEAD` and `POST`, with certain content types) are attempted without preceding checks. Other requests (e.g. `POST` with JSON or XML content, `PUT`, `DELETE`) are subject to a preflight `OPTIONS` request to determine whether or not the request is safe to send. This is mostly to [protect older servers][5] that might not be aware that browsers are now able to perform these cross-origin requests. Note that it is still necessary to take measures to protect against cross-origin `POST` requests made via AJAX, which have always been possible.

### CORS

[Cross-Origin Resource Sharing (CORS)][6] is a mechanism that allows site owners to selectively relax the same-origin policy by configuring their web server to set certain headers. Setting the `Access-Control-Allow-Origin` header to a given origin, for example, will enable the browser to make cross-origin requests from this origin.

For credentials (cookies) to be used in such requests, they must be explicitly included in the request itself, and the `Access-Control-Allow-Credentials` header must be set.

[1]: https://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf
[2]: https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-lekies.pdf
[3]: https://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection.html
[4]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
[5]: https://stackoverflow.com/questions/15381105/cors-what-is-the-motivation-behind-introducing-preflight-requests
[6]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
