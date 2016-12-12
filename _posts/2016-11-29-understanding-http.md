---
layout: post
title: "Understanding HTTP"
date: 2016-11-29
---

## Understanding HTTP

I wanted to write a HTTP server with ReactPHP, but first of all I had to understand how HTTP works.
Everyone uses an Apache or a nginx web server and some developers had stripped informations from a POST, GET or PUT request.
But did you ever written a HTTP request with your own hands? I didn't. I always used tools that do that for me (Browsers, HTTPie etc.), but
never did it with pure PHP. So wanted to give it try.

### HTTP request

So to send HTTP requests on my own I had to first understand how HTTP works. So a first looked on wikipedia but I wasn't statisfied with the information given me there.
So looked into the (RFC entry)[https://www.ietf.org/rfc/rfc2616.txt]. To be honest: I always hated to look into a RFC.
What maybe has something to do that english is not my native language.
But slowly I'm realizing that these are well written and in combination with the date of the publishment, these concepts will be little bit clearer.

I found what I was looking for how a HTTP request and HTTP response should, must and have to look like.
So first of all the header:
The header must be given. The header of a request consists of a starter-line consisting of the used HTTP version (HTTP/1.0, HTTP/1.1, HTTP/2.0 etc.) the method (GET, PUT, POST and other) and several header you can add if you want.

So in the end our HTTP request header can look like this.

```
GET /ip HTTP/1.1
Host: httpbin.org


```

That is all we need as header. (Note: The header field `Host` is a must header field in HTTP/1.1).
Easy isn't it?
But here are some specials. A header always and with double line break. A simple `\n\n` isn't enough you need to add a `\r\n\r\n` to mark the end of an header.
If we think in strings our request looks like this:

```
GET /ip HTTP/1.1\r\nHost: httpbin.org\r\n\r\n
```

And this is exactly how we can write this in a simple PHP program:

```php
// Establish TCP/IP connection to the server
$resource = stream_socket_client("tcp://httpbin.org:80", $errno, $errstr, 30);

// Send HTTP request
fwrite($resource, "GET /ip HTTP/1.1\r\nHost: httpbin.org\r\n\r\n");

// Read the response
$data = '';
while (!feof($resource)) {
    $data = fgets($resource, 1024);
    echo $data;
}
fclose($resource);

```

We sent our first own HTTP request. And what we reveive is a HTTP response from a server containing the IP we use in the internet

### HTTP response

A HTTP response is very similiar to the HTTP request. We have a header and a body. The starter-line of a HTTP response quiet different.
The line begins with used HTTP version followed by a status code and message for this code.

```
HTTP\1.1 200 OK

```

Like in the request the response can consist of several header fields and a body. The example above is a minimal response.
And like the header of the request the header ends with an `\r\n\r\n`.

### Identify the body

The identification of the body can be a little bit curios.
Nearly everybody in the web development heard about the `Content-Length` field in the header.
It describes what byte length the body of the response has. So the length of the content is known from the beginning until the end.

```
HTTP\1.1 200 OK
Content-Length: 3

bla
```

But did you hear about the field `Transfer-Encoding` with its `chunked` encoding? It is completly different approach. If Transfer encoding is set, little chunks containing the body will be send. One chunk consists of the size given in hexadecimal in the first line and the data. Each the size and the data have to end with ´\r\n´. The communication is completed when one of the communication partners ends the communication or when a chunk will be send with the size 0.

```
PUT / HTTP/1.1
Transfer-Encoding: chunked

3
bla
5
hello


```

As a string the message would look like this:
```
PUT / HTTP/1.1\r\nTransfer-Encoding: chunked\r\n\r\n3\r\nbla\r\n5\r\nhello\r\n
```

The transfer encoding have the problem that someone has to decode it. Normaly there is somekind of decoder on the server side.

That should be all you have to know about the basics of HTTP. There is far more just look into the RFC :-).
