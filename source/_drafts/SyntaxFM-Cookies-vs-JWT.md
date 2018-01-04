---
title: SyntaxFM-Cookies-vs-JWT
tags:
---

I listen to [Syntax.FM](https://syntax.fm). On their [recent episode](https://syntax.fm/show/026/all-about-redux-and-and-cookies-vs-jwt), Scott and Wes discuss "Cookies vs JWT". Having recently re-evaluated the pros and cons to handling user tokens via cookies or local storage, I thought it would a good time to document my findings and thoughts.

## TL;DR

- Don't store/send sensitive data in JWTs. 
- You can set JWTs in cookies or store JWTs in local storage.
- Sanitize/validate all data from the client before using/manipulating.
- Local storage solution is best for applications where clients may not be on the same origin.

## JWT (JSON Web Token)

Before we continue, I would like to clarify that both strategies can utilized JWTs to store data that has state such as expiration data, issue date, and a signature. If you're unfamiliar with JWTs I'd highly recommend playing around over at [jwt.io](https://jwt.io/). 

JWTs are strings have 3 parts that are dot-delimited. 1) a header has the algorithm used to signed the token and token type identifier. 2) a payload (where your data is) that has been JSON-stringified. 3) the verification signature result. All 3 parts are simply base64UrlEncoded. This is important, as that means all 3 parts are easily decoded and not inherently secure. The security comes from the signature that comes from the algorithm described in the header that is fead in the header and the payload along with a secret. The resulting output is unique. While this seems like nitty-gritty details, there seems to be a lot of misconceptions about JWTs that stem from these details.

## Cookies

User tokens/ids can be managed by the server by setting cookies on responses to logging in. In nodeJS using express you can set a response cookie simply with 

```javascript
function expressHandler (req, res, next) {
    res.cookie(COOKIE_NAME, COOKIE_VALUE);
    next();
}
```

where `COOKIE_NAME` is a string and `COOKIE_VALUE` is string or an object converted to JSON. Cookies can be signed by the server creating a "signed cookie" using a secret. Cookies can have an expiration date, an issued date, and other metadata. Subsequent requests will then have `req.cookies.COOKIE_NAME` (or `req.signedCookies` if its a signed cookie) given that the cookie is valid. You can also update cookies in the same fashion as issuing them.

## Local storage

User tokens can also be stored in the browsers local storage through client side code. Tokens can then be sent to the server on every request by setting a header. The common pattern is to set a default request that sends an `Authorization` header with the value `'Bearer USER_TOKEN'` if you have token from the server. This is just a [common pattern](https://tools.ietf.org/html/rfc6750).

## Issues

Doing a search for "local storage vs cookies" yields conflicting information most of which is regarding security. Both cookies and local storage can be "spoofed" and "sniffed". Because of this, use JWTs with a strong secret or at least a signed cookie with a strong secret. Validate every JWT signature and cookie signature before using the payload and you'll have an easy time.

Cookies pose their own issues as modern browsers only send cookies on requests to the same origin. This means that cookies are only sent to urls that share the same protocol, port number, and domain. This causes problems with client applications that make requests to apis on different domains, or even developing on a different port number than your api.

## Conclusion

Aside from origin issues with cookies, both cookies and local storage are viable, modern solutions for storing user tokens. Use JWTs or signed cookies. **Never store or send sensitive information in a cookie or JWT**.
