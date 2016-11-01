---
layout: post
title: "Understanding proxies"
date: 2016-10-31
---

I’m currently checking out how proxies work. There are many different explanations how the proxies work, but sadly not how a simple connection establishment for different kind of proxies work.

So I tried my best to explain it by myself.

## What is a proxy?

So first of all, I try to explain what a proxy is. A proxy lies between your client and a server. But why should I do something like this? Why shouldn't a communicate with a server directly? Well, sometimes you don't want that. Sometimes you want to hide your IP address or use a proxy so the website can't track your activities. There are many legal szenarios you can us with a proxy. So how does it work? Nothing changes for your client at all you just initate a communication to someone else: to the proxy. For the server nothing changes at all. The server just gets repsonses and will send a request. He just communicates with the proxy instead of the client.

As a client nothing really changes for you, you just say you want to use a proxy and the proxy will do the work.

## SOCKS proxy

What is SOCKS? SOCKS is protocol to send network packages between a client and server([Checkout Wikipedia](https://en.wikipedia.org/wiki/SOCKS)). So basically it forwards the requests and responses forward to their destinations.

Let’s look how this would work if I use a SOCKS Proxy between my computer and google:

The client establishes a TCP/IP connection to the SOCKS proxy. If this connection is succesfully established. The client have to send a connection request via [https://en.wikipedia.org/wiki/SOCKS#Protocol](the SOCKS protocol) containing the IP and port to google.com. The SOCKS proxy receives this connection request and initiates the TCP/IP handshake with google.com. In the case that this handshake is succesfull, the proxy sends a message to the client that connection is established. From this point the SOCKS proxy just forwards every request and response. We can now send a HTTP GET request to google.com. The Proxy forwards this requests to google.com and google.com answers with a response, which will be forwarded to the client.

![](/images/socks-proxy.png)

## HTTP proxy

A HTTP Proxy works like a SOCKS Proxy but a little different. The HTTP Proxy doesn't have the (obviosly) the SOCKS Protocol. So after the TCP/IP handshake between client and proxy, the client can send a HTTP GET request directly. The HTTP Proxy knows HTTP so the proxy knows that this request should be forwarded to google.com (because it is written in the HTTP GET request). 
So this is a little bit different from how the SOCKS proxy works, the SOCKS proxy just forwards every request because it's connection was established via the SOCKS protocol, while the HTTP proxy interprets every single request.

![](/images/http-proxy.png)

## HTTPS proxy

If we want establish a secure connection to google.com we can use a HTTPS proxy.
So what happens here? The client just uses the TCP/IP handshake as always. After this we say the proxy that we want to connect to google.com. The proxy establishes a TCP/IP connection to google.com and forwards this information to our client. After that our client connects via SSL/TLS to google.com. If the handshake was successfull we can send our requests and get responses completly encrypted and secure! 

![](/images/https-proxy.png)
