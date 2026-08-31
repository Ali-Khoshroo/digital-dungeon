---
title: Behind the Cookie 🍪: How HTTP Remembers You
date: 2026-08-31
draft: false
description: "HTTP has no memory — so how does a website know you're logged in? Cookies are one of the answers. Let's take a look at how cookies work under the hood."
tags:
- web hacking arsenal
- cookie
---

# What is HTTP Cookie🍪

As I've mentioned before in [A Tour of HTTP](/digital-dungeon/posts/introduction-to-web-and-browser/),HTTP is stateless,therefore it does not know about previous interactions with a client.In order to solve this problem,**HTTP cookies** are used.A **cookie** (also known as a web cookie or browser cookie) is a small piece of data a server sends to a user's web browser. Cookies enable web applications to store limited amounts of data and remember state information.

Cookies are mainly used for three purposes:

- **Session management**: For example user sign-in status, shopping cart contents, game scores, or any other user session-related details that the server needs to remember.
- **Personalizing**: User preferences such as display language and dark/light theme that user prefers .
- **Tracking**: Recording and analyzing user behavior.

## How is a cookie set

When a client first sends a request to the web server, the **server** responds with a `Set-Cookie` header containing a cookie value. The cookie value is then stored on the client’s browser.Then, every time the browser sends a request to the server it will submit the cookie(put it in the request), so the server will recognize the client(remember who we are).

#### Here is a simple example
1. The client sends a `GET` request to the server
2. The server responds with a `Set-Cookie`
3. The client will add the cookie to the requests it will send to the server

![](/digital-dungeon/Images/cookie_flow.png)


---

## Set-Cookie in details

An HTTP cookie is set based on **attributes provided by the server in the `Set-Cookie` response header**.

A cookie is set by specifying a `name-value` pair like this:
```http
Set-Cookie: <cookie-name>=<cookie-value>
```
For example, this HTTP response will tell the browser to store and set this cookies:
```http
HTTP/2.0 200 OK
Content-Type: text/html
Set-Cookie: favorite_weapon=banana_sword
Set-Cookie: theme=dark

[page content]
```

### Define scope of the cookie

The `Domain` and `Path` attributes define the **scope** of a cookie(what URLs the cookies are sent to).
Let's see an example:
```http
Set-Cookie: key=anyvalue; domain=example.com; Path=/search/
```

First let me mention that, the ("/") character is considered a directory separator, and subdirectories match as well. For example, if you set `Path=/posts`, these request paths match:
- `/posts`
- `/posts/`
- `/posts/xss/`
- `/posts/web/HTTP`

In this example, a cookie is set for the `example.com` domain and is specifically scoped to `/search/` path.

This means that the cookie containing “key=value” will be sent by the browser **only** when the requests are directed to URLs within the `example.com` domain and under `/search/` path.

Cookies are not bound to a single origin.Their scope is primarily controlled by the `Domain` and `Path` attributes. This means a cookie can be shared across multiple subdomains of the same parent domain.

### ❗A little history lesson

You may sometimes see cookies written like this:

```
Set-Cookie: theme=dark; Domain=.example.com
```

The leading dot (`.example.com`) comes from an **older cookie convention** where the dot was used to indicate that the cookie should also apply to subdomains.

Modern browsers no longer use the dot to determine this behavior. Today, these two are treated the same:

```
Domain=example.com
Domain=.example.com
```

Both allow the cookie to be sent to `example.com` and its subdomains.

So, the leading dot is mostly **historical syntax** that you may still encounter when reading older documentation, applications, or security reports.

❗But! The correct way to restrict and limit scope would be **NOT TO** include the domain attribute at all.Now you may be like:

![](/digital-dungeon/Images/icecube.png)

The reason for that is, if a cookie doesn't need to be shared with subdomains, it's better to **leave the `Domain` attribute out entirely**.
In that case, the browser creates a **host-only cookie**. This means the cookie is restricted to the host that set it rather than being made available to its subdomains.

For example, if `example.com` sets:

```
Set-Cookie: theme=dark
```

the cookie is associated with `example.com` and isn't automatically sent to:

```
sub.example.com
admin.example.com
```

This is generally preferable from a security perspective because it gives the cookie a **smaller scope**.

---

### Lifetime of a cookie

We can set an expiration date or an amount of time, so that after that the cookie will be deleted and will no longer be sent to the server.Depending on what is being set in the `Set-Cookie` header when cookies are being created, they can be **permanent** or **session** cookies.

#### Permanent
These cookies are deleted(expire) after the specific time that we set for them either by using `Max-Age` attribute or `Expires`.
Here is an example of both:
```http
HTTP/1.1 200 OK
Set-Cookie: session=abc123; Expires=Sun, 30 Aug 2026 18:00:00 GMT
```

```http
HTTP/1.1 200 OK
Set-Cookie: session=abc123; Max-Age=3600
```

#### Session cookie
**Session cookies** do not have an `Expires` or `Max-Age` attribute. They are generally removed when the browser session ends(The browser defines when the "current session" ends !), although modern browsers may restore session cookies when restoring a previous browser session.
But Of course, cookies can also be deleted before they naturally expire.

---

### Update cookie value

To update a cookie, the server can send a `Set-Cookie` header with the existing cookie's name and a new value(this can also be done using JavaScript). For example:
```http
Set-Cookie: oldcookie=new-value
```


---

### Other cookie attribute

I will cover some other cookie attributes here with a brief explanation of what they do and we will see them in a response to understand them better :

- **`Secure`** — The cookie is only sent over a secure connections, such as a TLS connection, preventing it from being transmitted over an unencrypted connection.
- **`HttpOnly`** — Prevents client-side JavaScript from accessing the cookie, helping protect it from **cookie theft through XSS**.


```http
HTTP/1.1 200 OK
Set-Cookie: session=abc123; Secure; HttpOnly
```



