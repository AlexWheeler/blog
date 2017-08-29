---
layout: post
title: "JavaScript Api Clients"
date: 2014-11-03 09:36:14 -0400
categories: javascript
---

Spending the summer working full time on the ChallengePost engineering team I
had the opportunity to ship a lot of really awesome features.  If you happen to
follow my blog you probably know that if there’s anything I spend a lot of my
time thinking about it is APIs.  Working at ChallengePost as an engineer is
awesome.  All feature development is managed on Trello.  If there’s a story
coming up you want to build you simply put your face on it and see if anyone
else would like to pair on it. One day, after finishing up a feature I was
working on, I noticed a new story from the product team pop up on our trello
board to implement the Find your faceobook friends and Github followers on
ChallengePost feature. I was super stoked.  After talking over my plan with
some team members we decided that the best idea would be to create our own
wrapper classes to work with the two APIs.  This would make them much simpler
to work with, as well as set a design standard for future integrations with
other services.  This experience got me thinking about all of the cool ways you
can build objects to talk with external APIs and is for sure a blog post of its
own.  In my last post I discussed finding and exploring undocumented APIs,
something I’ve found myself doing in a recent project.  In this post I’d like
to walk through building a simple API client to consume such an API.

![talking-computers.png](/assets/javascript-api-clients/talking-computers.png)

The single responsibility principle states that every context (class, function,
variable, etc.) should define a single responsibility, and that responsibility
should be entirely encapsulated by the context. Our context, the API client
will be the MbtaClient prototype from which we will instantiate our client
object.  This client will have one responsibility - to request data from the
[MBTA Realtime API](http://realtime.mbta.com/Portal/Content/Documents/MBTA-realtime_APIDocumentation_v2_0_1_2014-09-08.pdf).

The MBTA Realtime API is a standard RESTful API that exposes data to clients
through various URLs or endpoints.  Building a request is pretty
straightforward - start with the base URL

*http://realtime.mbta.com/developer/api/v2/*

and append the correct path and query string parameters.

For example:

appending *routes?api_key=wX9NwuHnZU2ToO7GmGR9uw&format=json* onto our base url

yields

*http://realtime.mbta.com/developer/api/v2/routes?api_key=wX9NwuHnZU2ToO7GmGR9uw&format=json*

which will return a JSON representation of all MBTA routes.

Since this base URL is the same across every request we’ll save it to a
variable baseUrl.  Now we can reference this variable instead of hard coding
the url every time we need it.  This will also save us a huge headache if the
MBTA decides to change the URL in future versions of their API since we only
need to update its value in one place as opposed to every occurrence throughout
our program.  One more thing to note is that every request must also contain a
valid api key.  Since it is only our program using this client it would make
sense to set this value at compile time, however we might want to allow others
to use our client so for this reason I’ve decided to add it as an argument to
the MbtaClient constructor so it can be passed in dynamically and set at
runtime.

```javascript
function MbtaClient(apiKey) {
  var baseUrl = "http://realtime.mbta.com/developer/api/v2/";
}
```

Great, our client knows about the base URL for generating requests.

Now before we write any more code we should take a moment to think about what
data we’re interested in.  For my current application I need to get current
positions of various vehicles.  Looking through the documentation it looks like
this data lives at */vehiclesbyroute* and requires a query string parameter of
route with a value of the route_id for said route. The full request would be:

*http://realtime.mbta.com/developer/api/v2/vehiclesbyroute?api_key=<api_key>*
&route= <route_id> &format=json

Now it is time to translate this request into a function we can call on our api
client.  The goal is to make retrieving and using data as easy as:

```javascript
mbtaClient = new MbtaClient("xxxxxxxxxx")

mbtaClient.vehiclesByRoute(<route_id>, function(data) {
  // do something cool with data
}
```


The first thing to figure out when examining this request is which information
is static across all requests and which information will change. Our client
should know about any static information and our custom functions should take
the dynamic information as arguments.  Let’s make sure we have all of the
static information covered.

```javascript
function MbtaClient(apiKey) {
  var baseUrl = "http://realtime.mbta.com/developer/api/v2/";
  this.vehiclesByRoute = function(routeId)  {
    var requestUrl = baseUrl + "vehiclesbyroute?api_key=" + api_key + "&route=" + routeID + format;
  }
}
```

Our client has:

1. baseUrl

2. api key

3. format (for this post we’ll assume we only want JSON responses, of course you could modify this)

Leaving us with 2 possibilities for dynamic content:

1. resource path (*/vehiclesbyroute*)

2. route_id

Seeing that the main point of this client is to limit complexity we want to
hide and abstract as much information as possible.  Since any request for a
vehicle’s location will use the */vehiclesbyroute* resource we can define this in
our `vehiclesbyroute()` function:

```javascript
function MbtaClient(apiKey) {
  var baseUrl = "http://realtime.mbta.com/developer/api/v2/";
  this.vehiclesByRoute = function() {

  }
}
```

*Side note: Chad Fowler has a great post on finding a balance between writing a
very concrete or abstract client and I highly recommend reading it.  In our
case we have abstracted the actual HTTP calls, but what I think is really cool
is that our functions can map 1:1 with the APIs interface.  in this case our
function vehiclesByRoute( ) requests resources living at the vehiclesbyroute
endpoint.  Just some food for thought so let’s continue.*

Therefore, the only dynamic value the we need is *route_id* - we’ll give
`vehiclesByRoute()` this information as an argument.

```javascript
function MbtaClient(apiKey) {
  var baseUrl = "http://realtime.mbta.com/developer/api/v2/";
  this.vehiclesByRoute = function(routeId) {

  }
}
```

And now the time has come.  All of the pieces are in place and we are ready to
build our request!  This will be accomplished in the function body.

First we build the full request URL including required query string parameters.

```javascript
function MbtaClient(apiKey) {
  var baseUrl = "http://realtime.mbta.com/developer/api/v2/;"
  this.vehiclesByRoute = function(routeId) {
    var requestUrl = baseUrl + "vehiclesbyroute?api_key=" + apiKey + "&route=" + routeID + format;
  }
}
```

Next we build our Ajax request using the [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) object, which will
allow us to asynchronously request information from the MBTA API.

```javascript
function MbtaClient(apiKey) {
  var baseUrl = "http://realtime.mbta.com/developer/api/v2/;"
  this.vehiclesByRoute = function(routeId, callback) {
    var requestUrl = baseUrl + "vehiclesByRoute?api_key=?" + apiKey;
    var request = new XMLHttpRequest();
    request.open("GET", requestUrl, true);
    request.onload = function() {
      callback(JSON.parse(this.response));
    }
    request.send()
  }
}
```

*note: Ajax and JavaScript callbacks resources in case you’re not familiar*

Let’s take a moment to walk through this code.  Line 6 instructs the request
object to send a GET request to the requestUrl.  The third argument, true,
tells him to make it asynchronous.

Since this request will be asynchronous we might not receive data right away.
Because of this line 7 says…Hey once you successfully receive our data do
whatever I tell you to do in that anonymous function after your equals sign.

Here is where things get really rad.  Callbacks are definitely one of the
coolest features of JavaScript so we’ll briefly go over them.  In JavaScript
functions are first class citizens.  This means they have all of the rights of
any other variables and because of this they can be passed into functions.
YES…functions can take functions as arguments.  With this we can do:

```javascript
client.vehiclesByRoute(<route_id>, function(data) {
  // do whatever we want with data
})
```

You might be wondering where that *data* parameter is coming from, well its
coming from line 8.  Line 8 says hey once you receive the data execute the
callback and make available to it the parsed JSON response.

Finally line 10 sends the request.

And that is it.  Of course we would want to handle errors and other server
responses, but that is out of the scope of this post.  Now we can simply
instantiate a new `MbtaClient` object and fetch vehicles locations from any route
by providing the route_id.

```javascript
mbtaClient = new MbtaClient("xxxxxxxxxx");
mbtaClient.vehiclesByRoute("57", function(data) {
  console.log(data);
});

mbtaClient.vehiclesByRoute("66", function(data) {
  // do something else with data
});
```

Want to dive deeper?

https://github.com/AlexWheeler/BuBus

Questions?

@askwheeler on Twitter
