---
layout: post
title: "Writing a HTTP server in ReactPHP"
date: 2016-12-16
---

A HTTP server in PHP? Nothing new. But a HTTP server written in reactPHP that is slightly new.
There are different aproaches to realize such server and here is mine.

## Writing a server in ReactPHP

Now we know how [HTTP](https://legionth.github.io/blog/2016/11/29/understanding-http) works we are
ready to write our own server with ReactPHP.

This means we are using streams to communicate between server and client.

### Thinking about streams

Using streams also means we can't rely that the data is always complete. 
An example:
We got the starter-line with the method and the HTTP version etc.,but the header isn't completed yet caused by network interferences.
Or the body isn't completed and we got only the half of it.
What I'm trying to say is, when you using streams you don't know when the data is completed, but this is ok because the RFC of HTTP does.
In the normal PHP world theses behavior is blocking, which means you wait until the HTTP request is complete. But we don't have time to wait
we are working with ReactPHP, so we don't have.
Imagine a HTTP request is send to a PHP application without ReactPHP. The whole application needs to wait until the whole request is completed
body **and** header.

Why wait for the body when you already have the header? That is one concept of the streams.

### Using response and request objects

First of all we think about how we want to use such a server.
We trying to create an API so the user decides how server should react to different requests. But how should a request and response look like?
We could just forward and receive strings, but it's 2016 we want to handle with objects. Luckily there is already a standard to think of HTTP
requests and responses: **PSR-7**. PSR-7 defines how a object should be build, which functions, methods and interfaces must, should and could be defined.

So let us think about it. We get a request as a string via our ReactPHP streams. We convert this string into an request object to work with it.
Sounds easy! And it is.

### Usage of the server

So how doest our server should look like when executed as user?
This could be a possible scenario.

```php
$loop = React\EventLoop\Factory::create();

$socket = new Socket($loop);
$socket->listen(10000, 'localhost');

$server = new HttpServer($socket, $callback);

$loop->run();
```
So we create a socket. So the server listen to a port. In this example port `10000`.
But what is this ominous `$callback` variable.
Well this is a function defined by the user of the this server. How he wants to handle the requests and which kind of responses should be returned.

```php
$callback = function (RequestInterface $request) {
    return new Response();
};
```

This example always returns a `HTTP 200 OK` message.

Well now we know how the user uses our code but how does our code look like?

### PSR-7 middleware

So we just need to use PSR-7 middleware. I decided to use `ringcentral/psr7`. There are many different, just run google and
check it out.

How does a request and response look like in this PSR-7 standard? 
Well very simple:

```php
$request = new RingCentral\Psr7\Request('GET', 'httpbin.org');
$response = new RingCentral\Psr7\Response(200, array('Content-Length' => 3), 'bla');
```

### The user code

Now we know what the user code must get and must respond. But how can user code use our API without breaking everthing?
Even that is simple(When you have the idea): a callback function.

That is very simple concept the callback function will get a request object and will return a response object.
The pseudo code would look like this:

```php
$callback = function(Request $request) {
    // you would check the request object and
    // would build a response object
    return new Response();
};
$server = new HttpServer($callback);
```

### Concept of the server

We know read much about stream, request/response objects and how the user of our server uses this server.

But how does it look like inside of the server. As seen in our chapter [about the usage of a server](#usage-of-the-server), we saw
this line of code:

```php
...
$socket = new Socket($loop);
$socket->listen(10000, 'localhost');

$server = new HttpServer($socket, $callback);
...
```

How does the constructor of our server looks like?

```php
class HttpServer extends EventEmitter
{
    public function __construct(ServerInterface $socket, $callback)
    {
        $this->socket = $socket;
        $this->callback = $callback;
        $this->socket->on('connection', array(
            $this,
            'handleConnection'
        ));
    }
...
```
Our `HttpServer` extends from `EventEmitter` which means it can handle different events.

If you already worked a little bit with ReactPHP you know that the second paramter is a method `handleConnection` refers to the first
parameter which is `$this`. Which is our current class. If you don't know you do :-D.

Now we have to define our `handleConnection` method.

```php
    public function handleConnection(ConnectionInterface $connection)
    {
        $connection->on('data', function ($data) {
            ...
        }
    }
```

So now our connection has a new event to listen on the `data` event. So if data will be send on this connection from the client to the server,
we can handle this data. Which will be our HTTP requests via stream.

I won't describe how you handle header and body here. Or how to differ between `Chunked-Encoding` and `Content-Length`, so
I will just put pseudo code in here.

```php
    public function handleConnection(ConnectionInterface $connection)
    {
        $connection->on('data', function ($data) {
            $response = $this->handleHeader($data);
            if ($headerCompleted) {
                $response = $this->handleBody($data, $request);
                if ($bodyCompleted) {
                    $connection->write(Psr7/str($response));
                    $connection->end();
              }
            }
        }
    }
```

This is it. We have a very simple HTTP server built with ReactPHP

If you want to know more about all this check out [my repository on github](https://github.com/legionth/php-http-server-react)
