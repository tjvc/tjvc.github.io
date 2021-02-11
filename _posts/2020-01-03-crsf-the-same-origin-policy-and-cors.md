---
layout: post
title:  "CSRF, the same-origin policy and CORS"
date:   2020-01-03
---

## CSRF

Using Cross Site Request Forgery (CSRF), an attacker can trick a user into making requests to a web service with which they have previously authenticated. This allows the attacker to perform destructive operations or extract data from the target service.

Assume a user has previously authenticated with Facebook. An attacker could trick this user into making requests to Facebook by clicking a link or executing some malicious client-side code, either on the attacker's own site or injected into another page. A classic example is embedding an image tag on a page visited by the user, with the tag linking to a protected resource on the target service. The browser will automatically attempt to resolve the link in the tag when loading the page, thereby performing the request without the user's consent.

## The same-origin policy

The origin of a web page is generally the scheme, host and port taken together. Requests to a different origin than that of the requesting page are known as cross-origin requests.

When a browser makes a cross-origin request as part of an AJAX call, it will raise an error and will not share the response with the calling code. This is the same-origin policy, which is intended to isolate content from different sources within the browser.

The policy prevents AJAX requests from being used to perform a CSRF attack that reads a user's data from a third party service. It prevents an attacker, for example, from serving a malicious script that could make `GET` requests to a service on a user's behalf and forward the responses.

'Simple' cross-origin requests (`GET`, `HEAD` and `POST`, with certain content types) are attempted without preceding checks. Other requests (e.g. `POST` with JSON or XML content, `PUT`, `DELETE`) are subject to a preflight `OPTIONS` request to determine whether or not the request is safe to send. This is mostly to protect older servers that might not be aware that browsers are now able to perform these cross-origin requests. Note that it is still necessary to take measures to protect against CSRF attacks (via AJAX or not) made using cross-origin `POST` requests, which have always been possible.

## CORS

Cross-Origin Resource Sharing (CORS) is a mechanism that allows site owners to selectively relax the same-origin policy by configuring their web server to set certain headers. Setting the `Access-Control-Allow-Origin` header to a given origin, for example, will enable the browser to make cross-origin requests from this origin.

For credentials (cookies) to be used in such requests, they must be explicitly included in the request itself, and the `Access-Control-Allow-Credentials` header must be set.

## Other CSRF countermeasures

HTTP methods should be used as intended. Specifically, use `POST` not `GET` for state-changing operations. Assuming that this is the case, non-AJAX `GET` requests can be considered safe, as even if they are unintended, the response is just returned harmlessly to the user's browser.

`POST` requests can be protected by including a security token, verified by the server, in forms and AJAX requests, as happens in Ruby on Rails.

`GET` requests for dynamic JavaScript resources can be vulnerable, as an attacker could include such a script in a `<script>` tag (exempt from the same-origin policy) on her own page, and potentially extract user data after the script has executed (e.g. from a global variable). Rails prevents embedding of JavaScript responses by default for this reason.

## References

* <https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS>
* <https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy>
* <https://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf>
* <https://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection.html>
* <https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-lekies.pdf>
* <https://stackoverflow.com/questions/15381105/cors-what-is-the-motivation-behind-introducing-preflight-requests>
