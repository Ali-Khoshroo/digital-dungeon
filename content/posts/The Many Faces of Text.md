---
title: The Many Faces of Text
date: 2026-08-28
draft: false
description: "Ever wondered how text, symbols, and special characters travel across the internet without getting lost? This guide breaks down the world of encoding"
tags:
- web hacking arsenal
- encoding
---
# What is encoding ?

Your computer only understands `1` & `0`.So how can it understand, save or display different characters, if it doesn't know what they are?
It's like someone talking Chinese to me(i'm saying this only cause i don't understand Chinese).I would be like:

![](/digital-dungeon/Images/confused_meme.jpg)

What Help's your computer to understand a character like `A`, is called **Encoding**.Encoding is an **agreement**, it's a **mapping between numbers and meanings**.

For example let's say we agree to represent `A`, with this `01000001`.
Then when your computer see's `01000001` it will know it is `A`.This is "decoding" (ASCII -> Text)

![](/digital-dungeon/Images/Encoding-example.png)

But we have 2 problems:
1. We have to do this for all letters in alphabet,digits,special and characters.
2. How can i know that my friend's computer will have the same mapping as mine has.How would i know that it doesn't think `01000001` is `T` ?
So all the computers should agree on the same thing.That is called an **Standard**.A standard that computer manufacturers, designers, and programmers agree to adhere to.

### Let's have an example

ASCII(American Standard Code for Information Interchange) character encoding.ASCII is used by programming languages for representing characters and symbols in source code and is a fundamental in data transmitting protocols.

Here is a part of the ASCII table for you so you will get the idea :

![](/digital-dungeon/Images/ASCII_table.png)

So if i want to say `hi` to you in ASCII , i would say `104 105`.Lowercase _h_ is represented by the ASCII code 104(decimal), and lowercase _i_ is represented by the number 105(decimal).

ASCII characters are represented in binary, providing a machine-readable format that computers use for internal processing.

❗Keep in mind : Encoding is different from `Hashing` or `Encryption` we will not cover those in this post, just know that they shouldn't be taken as equals.



---
Now that we are familiar with what encoding is, Let's go back to the main topic.

# URL Encoding

Web browsers request pages from web servers by using URL.You have definitely  seen one, they are something like this : `https://example.com`.

URLs can only be sent over the internet using ASCII characters.But most of the time URLs contain characters that are not in ASCII character set,so it has to be converted into a valid ASCII format. That's where **URL Encoding** comes in.
URL encoding replaces any unsafe ASCII character with `%`+`hexadecimal digit`.Also URLs can not contain `space` so they are replaced by `+` or `%20`.

Here is an example, let's say we have this phrase `<hello> world` after URL encoding it, it will turn into `%3Chello%3E%20world`


# Double Encoding

In double encoding characters are URL encoded twice.First they are encoded to a `%hex` form, then when it's going to be encoded for the second time, the encoding is applied to the `%`.
Some common double encoded characters:
```
Character | Single  | Double
----------|---------|--------
.         | %2e     | %252e
/         | %2f     | %252f
\         | %5c     | %255c
<         | %3c     | %253c
>         | %3e     | %253e
'         | %27     | %2527
"         | %22     | %2522
```

## Attack scenario 🎯

This technique can be used to bypass security controls or cause unexpected behavior from the application.

Imagine you're testing a web application that lets users access files through a URL:

`https://target.com/download?file=report.pdf`

You notice that the application is vulnerable to **path traversal**, but there's a WAF sitting in front of it. The WAF is configured to block requests containing `../../../etc/passwd`.

You try the obvious payload : `../../../etc/passwd`

The WAF detects it and blocks the request.You then try URL encoding : `%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd`

The WAF decodes it once, sees `../../../etc/passwd`, and blocks it again.
But then you notice something important: **the WAF and the application don't necessarily decode the request the same number of times.**

So you encode the payload **twice**:

`%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd`

The request reaches the WAF. After decoding it once, the WAF sees:

`%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd`

It no longer matches the WAF's `../../../etc/passwd` rule, so the request is allowed through.

The application then decodes the parameter **again**:

`../../../etc/passwd`

The original malicious path has effectively reappeared **after the security filter has already made its decision**.

This is the basic idea behind a double-encoding bypass: **the attacker hides the payload from a filter by encoding it one layer deeper than the filter expects.❗**


---

# HTML Encoding

In HTML, certain characters have certain meanings and if they are not handled correctly they can be dangerous.For example characters `<` and `>`can represent opening and closing of an HTML tag. We should make sure that these characters are  displayed and treated as a text(in textual form) rather than HTML syntax.In order to do that we need to HTML encode them.How can we do that?

To HTML encode a string we should replace each reserved character with its HTML entity.For example:
```
& -> &amp;
< -> &lt;
> -> &gt;
" -> &quot;
' -> &#39;
```

The browser then renders those entities as the **original characters** on screen without treating them as tags or syntax. This one operation is an important XSS defense, and it is also what lets you show code snippets, math symbols, and user comments inside a web page without breaking it.

Let me give you a pointer !
HTML entities come in two forms: named references like `&lt;` and numeric references like `&#60;` or `&#x3c;`. Both resolve to the same character.

![](/digital-dungeon/Images/table.png)

Also there might be a confusion between HTML encoding and URL encoding.Remember that HTML encoding uses entity references (`&lt;`) to make characters safe inside an **HTML document**. URL encoding, also called percent-encoding, uses a percent sign followed by hex digits (`%3C`) to make characters safe inside a **URL**.


---

# Base64 Encoding

First let me make something clear,cause it is commonly confused by many.Base64 is **NOT** an encryption scheme.It doesn't encrypt,it encodes!

Why is it called "Base 64"?
Base64 is a binary-to-text encoding scheme that represents data in ASCII string format and then **carries that data across channels**.

Base64 is considered a radix-64 representation. The encoding scheme breaks binary data into 6-bit segments, which are then mapped to one of 64 characters(we will see what are these 64 characters soon) to ensure that humans can read the printable data (2⁶ = 64 characters).

This means each base64 character is 6 bits of data.
Base64 is an encoding and decoding method that represents binary data based on 64 printable characters.
These 64 printable characters include:
- 26 uppercase letters A — Z
- 26 lowercase letters a — z
- 10 digits 0–9
- plus two other characters `+` and `/`.


## Now let's see how it works

Ok! let's start with this.`1 byte` = `8 bits`
So `3 bytes` will be `24 bits`.That is perfect for us cause `4 * 6 = 24`.Based ob this information, if the number of bytes we have is multiple of 3,we are happy as we can be.

But what if that's not the case?Bye Bye base 64? NO!

Here comes the "Padding".If we have any missing bit we will use padding.So we look at data `3 bytes` at a time until we have either 1 or 2 bytes left.
Now if we have `1 byte` left, it means we will use `6 bits` of it and 2 will remain we will add 4 `0 bits` to the end of it so it becomes `6 bits`.Now we have two 6-bit groups, which give us two Base64 characters. Since we only had 1 byte of actual data, we add two `=` signs as padding(it might sound confusing but trust me it's really easy once you see an example which i will provide down below).

If we have `2 bytes` left we will have `16 bits`, that we will only use `12 bits` of it , so this time `4 bits` will remain.In order to make them 6 we will clearly add 2 `0 bit` to the end of our binary and Now we have three 6-bit groups, which give us three Base64 characters but because we only had 2 bytes of actual data, we add one `=` sign as padding
Here is an example:
![](/digital-dungeon/Images/base64-encoder-flow.png)



---


# Unicode Encoding

Unicode is a universal character standard designed to represent characters from languages and writing systems around the world. Each character is assigned a unique code point. Unicode itself is not an encoding; UTF-8, UTF-16, and UTF-32 are different ways of encoding Unicode code points.

As we learned before Computers ultimately store information as numbers, and we used ASCII for communicating with the computer.But what about other languages in the world.ASCII will not work for something like Persian language.What about an emoji 😥?

Unicode was created to provide a common system for representing characters from languages and writing systems around the world.
Think of **Unicode as a huge catalog of characters**.Each character in this catalog is assigned a unique number called a **code point**.
For example:
```
A → U+0041
B → U+0042
é → U+00E9
```
The `U+` simply tells us that we're talking about a Unicode code point, and the number after it is written in **hexadecimal**.
