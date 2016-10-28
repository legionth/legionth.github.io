---
layout: post
title: "Using proxies in PHP"
date: 2016-10-28
---

So tried to establish a connetion to a proxy via PHP, but I couldn't find an example without using curl or ohter third libraries. I just wanted the to use the PHP internal functions, so no further magic. 

So first of all I tried my luck with the SOCKS proxy:


```php
    $resource = stream_socket_client("tcp://<proxy-ip>", $errno, $errstr, 30);

    if (!$resource) {
        echo 'No connection to proxy';
        return;
    }

    $host = gethostbyname('www.httpbin.org');
    $ip = ip2long($host);
    $port = 80;
    $socks = pack('C2nNC', 0x04, 0x01, $port, $ip === false ? 1 : $ip, 0x00);

    fwrite($resource, $socks);

    while (!feof($resource)) {
        $data = fread($resource, 8192);
        echo $data;
        if (strlen($data) == 8) {
            fwrite($resource, "GET /ip HTTP/1.1\r\n");
            fwrite($resource, "Host: www.httpbin.org:80\r\n\r\n");
        }
    }
```
The function `stream_socket_client` just open a TCP/IP connection to our proxy server. In the next step we prepare the socks protocol to send to the proxy server. We enter here the ip(!) and the port of our destination server(httpbin.org). This data package will be send to the proxy and the proxy will establish a TCP/IP connection to the server. The proxy returns code when this connection is succesfully established. In this case it is a string of the length '8' or hex value of '0x5a'. Now we can send our HTTP GET request to the server and get a response from that server.

The HTTP Proxy looks nearly the same:

```php

    $resource = stream_socket_client("tcp://<proxy-ip>", $errno, $errstr, 30);
    
    if (!$resource) {
        echo 'No connection to proxy';
        return;
    }

    fwrite($resource, "GET /ip HTTP/1.1\r\n");
    fwrite($resource, "Host: www.httpbin.org:80\r\n\r\n");

    while (!feof($resource)) {
        $data = fread($resource, 8192);
        echo $data;
    }

```
The connection to the HTTP Proxy is equivalent to the SOCKS proxy. Except we don't need the socks protocol. We can send our HTTP GET requests immediatly.

The HTTPS proxy is little bit more complicated then the SOCKS or HTTP proxy:

```php
$resource = stream_socket_client("tcp://97.77.104.22:80", $errno, $errstr, 30);
stream_context_set_option($resource, 'ssl', 'peer_name', 'httpbin.org');

if (!$resource) {
    echo 'No connection to proxy';
    return;
}

fwrite($resource, "CONNECT www.httpbin.org:443 HTTP/1.1\r\n\r\n");

while (!feof($resource)) {
    $data = fread($resource, 8192);
    echo $data;

    if ($data == "HTTP/1.1 200 Connection established\r\n\r\n") {
        if(!stream_socket_enable_crypto($resource, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT)){
            exit();
        }
        fwrite($resource, "GET /ip HTTP/1.1\r\n");
        fwrite($resource, "Host: www.httpbin.org:443\r\n\r\n");
    }
}

```
As always we create a TCP/IP connection to the proxy server. But in the next step we have to a additional option to our stream. We eed to seed the 'peer_name' to 'httbin.org'. On a normal connection between server and client the 'peer_name' would always be the server name. But we use a proxy here. So we need the name of the original server(httpbin.org). If TCP/IP connection between client and proxy is established, we send a HTTP CONNECT request to signal that we want to create a SSL/TLS connection to the server. If the response from the server is a '200'-code we can now establish the secure connection. With the function 'stream_socket_enable_crypto' we create a SSL/TLS connection from the client to the server. If the SSL/TLS handshake is successfull we can send our HTTP GET request, now encrypted and secure. The proxy server only sees enrypted communication between our client and the server.