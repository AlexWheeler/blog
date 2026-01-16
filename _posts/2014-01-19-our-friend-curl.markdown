In my previous post I shared my thoughts on the emergence of tens of thousands of APIs that companies are racing to develop in an attempt to see what applications people will be able to build on top of their data or services.  In this post I’d like to share a useful tool known as cURL which allows us to design a request, send it to a server, and inspect the response.

Most modern APIs follow a RESTful architecture.  REST stands for representational state transfer.  Put simply RESTful architecture makes use of the web’s hypertext transfer protocol’s four main request types, GET PUT POST DELETE on resources.  You can best think of these HTTP requests as verbs acting on some resource (noun) at a given location (server.)  URLs or the things you type into the address bar in your web browser simply direct users to a certain resource.  So when we navigate to a url such as my twitter handle:

![twitter](/assets/twitter.png)

What we are really doing is sending an HTTP GET (hence the https://) request to the host at twitter.com requesting a resource - the profile for the user ‘askwheeler’.

So, what happens when we want to incorporate this user data and some other data that a given company might have, into an application of our own?  As we learned earlier we use an API if available.

The most common ways you will be interacting with an API are probably ways you’re already familiar with interacting with any other website, by POSTing and GETing data to and form their servers.  Sometimes these requests are as simple as requesting a given url, but sometimes APIs require you to send somme extra info along about yourself, what info you need, and the format you would like to receive it in – remember our library example from my last post?  Before you make a request you’ll want to make sure you’ve configured your request correctly and that you are receiving the expected response.  This is where cURL comes in handy.  Commands are very simple and run directly from the command line.  You begin by typing curl, follow it by some options, and end it with a url.  We will use http echo service for our endpoint, which is a site that provides a JSON representation of the response.

GET

The most common request is a GET request and this is what cURL defaults to. So if we send:

```curl
curl http://echo.httpkit.com
```

It will return our response:

![httpkit-get](/assets/httpkit-get.png)

As you can see we simply sent a GET request asking a host “echo.httpkit.com” for the uri located at ’/’

However we usually want to send more than this so the next thing you are able to do is attach some query string parameters as so:

```curl
curl http://echo.httpkit.com?key=value
```

which returns a response of:

![httpkit-kv](/assets/httpkit-kv.png)

You can see that we just sent a GET request to the same host and passed along the parameter “key” with a value of “value”

Great so we can GET data from the server, but sometimes when working with APIs you need to POST data to the server, DELETE data from the server, etc.  This can easily be done with cURL using the -X plus the name of the method you want to use followed by your request.

```curl
curl -X POST echo.httpkit.com
```

which returns:

![httpkit-post](/assets/httpkit-post.png)

As you can see now the method is a POST request.

Other times we want to provide the host server with some meta data either about ourselves and our request, which can be done with the -H followed by the request.

```curl
curl -H "Authorization: Key9999999999" http://echo.httpkit.com
```

which we can see will return:

![httpkit-headers](/assets/httpkit-headers.png)

Usually when you are POSTing or PUTing data to a server you will be including data as well as expecting the server to return a certain format, which we can do by combining -X -H and -d (for data).

```curl
curl -X PUT -H 'Content-Type: application/json' \
-d '{"name": "Alex", "favoriteColor": "blue"}' http://echo.httpkit.com
```

Which returns:

![httpkit-put](/assets/httpkit-put.png)

This is just a basic intro to all of the possibilities with cURL command, but definitely a great way to get you started on your way to toying with some HTTP requests and working with APIs.  You can find more information at:

[HttpKit](http://httpkit.com/resources/HTTP-from-the-Command-Line/)
