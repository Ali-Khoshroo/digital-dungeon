---
title: Breaking the Cookie Jar
date: 2026-09-05
draft: false
description: "Cookie tossing and cookie bombing are often overlooked—but chained with the right weakness, these tiny bugs can pack a serious punch."
tags:
- web hacking arsenal
- cookie
- cookie vulnerability
---
# Cookie Tossing

When an attacker gains control over a subdomain, for example through an XSS vulnerability or by design (such as a service that creates subdomains for customers), they can set cookies on the **parent domain**. This might not seem dangerous at first, but it can be exploited by setting the attacker's session cookie on the victim's browser for specific endpoints.

Basically attacker can fix his/her session with victim's on a specific path.So for example if attacker fix his/her cookie for `/card-info` you will be sending info using attacker's session,which will be like if you are logged in to attacker's account and doing what ever you are doing , so in this example,your card info will be added to attacker account.

When finding an XSS on a seemingly useless and not so important subdomain,this can help you to raise the impact.

---
## How Browsers Identify Cookies

Let me explain what is going on.First of all browsers identifies cookies by combining `name + domain + path`.So it's totally possible to have 2 cookies with the _exact_ name.
We can have multiple cookies with the same name, as long as we have different path or domain.

For example:

```
Set-Cookie: session=AAA; Domain=example.com; Path=/
Set-Cookie: session=BBB; Domain=example.com; Path=/admin/
```

The browser can store both:
![](/digital-dungeon/Images/stored_cookie_table.png)


Now consider these two requests.
If the user visits:

```
https://example.com/
```

only the cookie whose path matches `/` is sent:

```
Cookie: session=AAA
```

But if the user visits:

```
https://example.com/admin/
```

both cookies match the request:

```
Cookie: session=BBB; session=AAA
```

Notice something interesting here: the browser does not necessarily choose only one cookie.Both cookies can be sent when both match the request.

### The longest path comes first
Now that we have multiple cookies,what is going to happen?do we have any specific order? YES!

When multiple cookies with the same name match a request, cookies with a **longer path are ordered before cookies with a shorter path** when constructing the `Cookie` header.

❗ This does **not** mean that the browser is overriding or deleting the other cookie. Both cookies still exist and both may be sent.
The important part is that the **server receives the cookies in the** `Cookie` **header and has to parse them**. If there are duplicate cookie names, how the application/framework handles them can become important.

For example:
```
/admin/   → longer, more specific path 
/         → shorter, less specific path
```

### Cookies and Subdomains

If a sensitive cookie is scoped to the parent domain:
```
Set-Cookie: session=AAA; Domain=example.com
```

then that cookie is available across the domain's subdomains.
So if you have:

```
www.example.com
admin.example.com
blog.example.com
```

the cookie can potentially be sent to all of them.
This is one reason why sensitive cookies should generally be scoped as narrowly as practical.

If i want to explain the flow of the Cookie Tossing really briefly it's something like this:
You drive user to the subdomain you control(or have compromised), set a same-named cookie scoped to the parent domain from there, and when the victim later visits the parent domain, your cookie rides along — potentially alongside or instead of the legitimate one.

# Attack patterns
## 🍪 Secret Skimming

### Where you can use it
- file upload
- credit card info
- internal messages
- access tokens

### Pattern
1. Affect a "Submit" endpoint
2. Any secret is a CVSS impact
3. Steal or leak sensitive data from victim


## 🍪 Privilege Hijacking

### Pattern
1. Affect a privileged endpoint
2. Unaware victim will initiate a privileged action
3. Apply victim's privileges to your own account


---

## How to find it

Before starting to test,we are looking for a setup like this:

- **The endpoint relies on cookie-based authentication.** Obvious, but worth confirming first — session tokens, not just auth headers or bearer tokens.
- **The session cookie doesn't rotate on every request.** If the token regenerates on each call, you won't be able to reliably tell whether your injected duplicate had any effect, since the "real" value keeps changing underneath you.
- **The cookie is scoped to the parent domain (not host-only).** As covered earlier, this means no explicit narrow scoping — if `Domain=example.com` is set rather than left blank, the cookie is fair game from any subdomain you control.
- **The server/app tolerates duplicate cookie names without erroring out.** Some frameworks reject or choke on malformed `Cookie` headers — you need one that just picks one value and moves on.

```
The core question you're trying to answer is: when duplicate cookies exist, which one does the server actually trust?
```

## Steps

1. Send a request to an authenticated endpoint, then remove cookies one at a time and resend. When removing a specific cookie breaks authentication, you've found the one that matters.
2. Add a second cookie with the _same name_ as the session cookie but a garbage/meaningless value, placed _after_ the real one in the `Cookie` header:`Cookie: session=REAL_VALUE; session=garbage`
	If the response is still `200 OK`, the server tolerates duplicates and — most likely — reads the _first_ occurrence.
3. Move the meaningless cookie to first place and look for `200 OK`.

## What it can work on(attack vecotrs)
1. XSS on a subdomain
2. Cookie/header reflection
3. Subdomain takeover
4. Self Xss


# Cookie Bombing

The Cookie Bomb is a clever exploitation of the trust relationship between the browser and the server.It won't steal your data, but it can still cause real damage,it can disrupt services and lock out users.

A "Cookie Bomb" attack happens when an attacker finds a way to make a user's browser store a huge number of cookies, or cookies with unusually large values.

Most modern browsers allow up to about 4KB per individual cookie, and roughly 50 to 180 cookies per domain (the exact numbers can be different based on the browser you are using). Meanwhile, web servers(like Apache, Nginx, or Node.js)have their own limit on how large the total size of all HTTP request headers can be. That limit is usually somewhere between 8KB and 16KB. If a request goes over this limit, the server rejects it outright, usually with an HTTP 431 error ("Request Header Fields Too Large") or an HTTP 400 error ("Bad Request").


## The lockout

Once an attacker manages to plant enough cookies, every future request the victim makes to that site will automatically include all of them, packed into the `Cookie` header. Once that header's total size crosses the server's limit, the server stops accepting _any_ request from that user — locking them out of the site entirely.


This is a "client-side" DoS because it doesn't crash the server for everyone.It only breaks things for the one user who got "bombed".But if an attacker can automate this against many users at once, or specifically target admins, the damage can scale up fast.

## How it's delivered

Attackers typically pull this off through an XSS vulnerability, or by hosting a malicious script on a shared subdomain — either method can be used to sneak the oversized cookies into the victim's browser.

