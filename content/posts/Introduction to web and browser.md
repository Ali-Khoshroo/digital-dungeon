---
title: A Tour of HTTP
date: 2026-08-26
draft: false
description: "Take a journey through HTTP, from requests and responses to headers, methods, status codes, and everything in between."
tags:
- web hacking arsenal
- http
---
⚠️ Spoiler Alert: in some parts we may take a little detour in our journey, getting little bit deeper in some subjects.

# What is HTTP

HTTP (Hyper Text Transfer Protocol) is the protocol that runs the world wide web.At a fundamental level it is based on a *client-server* architecture.Usually the client requests content (typically via browsers) and the server/application delivers the response.
The **default** port for transmitting HTTP is **TCP port 80**, but it can also operate over different ports.

### Some of HTTP properties

- HTTP is a **Stateless protocol**.What does that mean? HTTP is kind of stupid, it does not have any idea about what is going on between client and server.That means that since it has no memory or knowledge about the communication between client and server , two requests don't have any relation to each other (to manage states such as login **cookies** are used).
- HTTP is transmitted as **Plain Text** over the network, it is an unencrypted protocol.This means that devices on network such as routers or proxies are able to read and modify the traffic.
  **HTTPS** is the solution.it encapsulates HTTP within an encryption layer , TLS/SSL (TLS is the newer, more secure version of SSL, providing enhanced encryption and integrity checks.)
- Since HTTP is getting transported over TCP it is reliable.(because of TCP)

### Communications

HTTP communications happen in two parts:
1. HTTP request  (client sends this asking for a resource)
2. HTTP response (server sends this as the response)

Lets see an example.The following request is asking for the "index.html" file hosted on example.com :

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.
212 Safari/537.36
Referer: https://google.com
```

Lets get into detail for what each line is:

### GET /index.html HTTP/1.1
**GET** is an HTTP request method , used to request data from a specified resource.Here the resource is "index.html".Followed by the version of the HTTP protocol, here it is `HTTP/1.1`.

[What is HTTP request method?](#http-request-method)
### Host
Host field is telling us who are we asking for the resource(who request is being submitted to).So in this example we are asking `www.example.com`.
### User-Agent
It will allow web server to identify the Operating System (OS) and browser of the client making the request.This information helps websites serve content tailored to different browsers and operating systems(for example Websites often serve different layouts to mobile and desktop browsers).
	
If you look at some HTTP requests, while using Chrome, Edge, Safari, Opera, etc.You may see that the `User-Agent` still has "Mozilla" in it.
**What on earth is going on? What is happening?**
Early browsers used `User-Agent` strings to identify themselves. **Netscape Navigator**, whose development was associated with Mozilla, became extremely popular, and websites started doing things such as:

if `User-Agent` contains "Mozilla" : send advanced version of the website
else : send simpler version
	
This started to create compatibility problems for other competing browsers.So they started to include `Mozilla` , even though they weren't Mozilla.

if you look at a request made by a Chrome browser you will see something like this :
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
AppleWebKit/537.36 (KHTML, like Gecko)
Chrome/143.0.0.0
Safari/537.36
```
When you first see that, it makes no sense.Which one are you ? Do you have some kind of identity crisis? Is Chrome all four of these? **NO**
These tokens have accumulated over decades for compatibility.
### Referer
This header will tell the server the URL of the web page where the user is coming from.
For example, if the `Referer` header shows `www.google.com`, it means that the user arrived at the current website by clicking a link on Google search.

---

This HTTP request will have a response like this:

```
HTTP/1.1 200 OK
Date: Sun, 26 Nov 2023 12:00:00 GMT
Server: Apache/2.4.41 (Unix)
Content-Length: 450
Content-Type: text/html; charset=UTF-8
Connection: close

<html><body> <h1>Hello from example.com</h1></body></html>
```

Now lets get deep in this one.

### HTTP/1.1 200 OK
This field shows that the "HTTP 1.1" protocol is being used and the request has been successfully processed as indicated by the HTTP status code of 200.

[what is HTTP status code?](#HTTP-status-code)
### Date
This field shows the timestamp when the response was sent.
### Server
This field shows that the server is running `Apache 2.4.41` and is hosted
on **the Unix operating system**. Revealing this field can potentially help
attackers, but it is not necessary(it optional) and can be removed or even
replaced with a fictitious value.
### Content-Length
This field specifies the size of the response content in bytes; in this case, it is "450 bytes".
### Content-Type
This field indicates the type of content being sent (HTML) and the character encoding being used, that is, “UTF-8”.
Some other content types:
```
Content-Type: image/png
Content-Type: image/jpeg
Content-Type: video/mp4
Content-Type: application/javascript 
Content-Type: application/pdf
```

### Connection
This field allows the sender to specify options that are desired for that connection.
Most commonly this header can have 2 values :
- keep-alive :This directive indicates that the client wants to keep the connection open or alive after sending the response message(In HTTP 1.1 version, by default uses a persistent connection where it doesn't close automatically after a transaction).
- close :This directive indicates that the client wants to keep the connection open or alive after sending the response message(In HTTP 1.1 version, by default uses a persistent connection where it doesn't close automatically after a transaction).


#### what is HTTP status code ? {#HTTP-status-code}
HTTP response codes are represented by three digits, indicating the status of the response. Each status code represents different response category.

![image](/Images/http-status-codes.jpeg)


#### What is HTTP request method ? {#http-request-method}

![image](/Images/http_methods.jpeg)

The most essential ones are `GET` and `POST`. While these methods are commonly used in web interactions,other methods are optional and may serve specific purposes.
`GET` is traditionally used to retrieve content, and `POST` is used to submit content to the
server.
However, GET can also be used to send content to the server.like this:
```
GET /login.php?username=myusername&password=mypassword
HTTP/1.1
Host: www.redseclabs.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.
4430.212 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Content-Length: 34
Connection: close
```
But this will have security problems, for example:
`GET` requests are logged in server logs, hence anyone with access to
these logs, such as unauthorized users exploiting security vulnerabili
ties, could see the complete URL. This becomes a concern when end-
points inadvertently leak logs or when there’s unauthorized access to
the logs beyond intended administrative oversight.
