---
layout: post
title: "Hacking Your Environment: Exploiting Undocumented, internal APIs, Part I"
date: 2014-09-08 09:36:14 -0400
categories: javascript api
---

I’ve recently found myself obsessing over the seemingly infinite ways to hack
my environment by extending and improving upon applications I use everyday.  As
a college student I find myself using gmail quite often for anything from
chatting with professors, to more recently, getting a hold of friends while my
phone is broken.  As a developer I tend to use many keyboard shortcuts, both on
the web and while crafting code.  When I realized that Gmail’s web version had
no logout shortcut I built a simple fix called Gkey, which allows users to
logout of gmail by pressing *ctrl-L* - free to download via [Chrome Web Store](https://chrome.google.com/webstore/category/extensions).
This is just one example, but can help get you thinking about ways to extend
your apps’ functionality.

Adding a feature or two to a web application by injecting javascript into a
page can be really fun, but don’t forget that we are developers, hackers - the
architects of the future.  We possess the knowledge and tools to truly change
the universe.

Universities provide many awesome things for students - dorm rooms, cafeterias,
classrooms, even gyms.  The problem is that many of these goods are durable and
can be hard to improve.  Your dorm room is too small? You could knock down the
wall into your neighbors room, thus making it a penthouse, but I don’t think
that would go over well with the housing department.  You know lunch in the
cafeteria would be more enjoyable if they provided Tapatio instead of Cholula
hot sauce (me too)?  You could always bring some for yourself and maybe you
friends, but this would prove pretty difficult and expensive to scale to the
rest of your school, especially if it has 33k+ students such as Boston
University where I attend.  Besides tangible goods, schools also provide many
software solutions to existing problems.  If you’ve attended university
recently you have most likely browsed and chosen classes via some online
application.  You might have also recently used services such as printing via
software your school provides.  Just as you did with your dorm room or
cafeteria, I’m sure you or your friends have thought about ways these services
could be vastly improved.  Lucky for us these services are all digitcal and run
on code written by people no smarter than you or me.  With just a little bit of
digging I’m sure you can find ways to leverage their existing architectures to
make them behave how you would have designed them had your school paid you -
most likely very, very large sum of money.

In this post I’d like to go over a recent, ongoing project of mine that
involves creating a new and improved BU Bus app using existing (non-public)
APIs.  The BU Bus is a free shuttle that takes students around campus.  The
existing [Bu Bus App](http://www.bu.edu/thebus/live-view/) shows students in real time where the buses are and the
various stops they can get off at.  The problem with this is that there are
many more ways to get to classes, including MBTA buses, trains, Uber, etc.  If
all of this data is available then there’s no reason students shouldn’t be able
to compare all of their options, ranked fastest to slowest, and chose which
method suits their schedule best when heading to class.  This is the problem
I’ve set out to solve.

*Existing live view of the BU Bus app*

![live-view](/assets/hacking-your-environment/live-view.png)

The first you’re going to want to do when thinking about building on top of
another platform is to use the app as you normally would and simply figure out
the basic architecture.  The first thing I notice when looking at this live
view is that the app is using a Google Map to display the bus locations.

![live-view](/assets/hacking-your-environment/live-view.png)

We know this by noticing *Google* in both the bottom left and bottom right
corners of the map.


![live-view](/assets/hacking-your-environment/google.png)

Having worked with Google Maps before I know they have awesome APIs for
plotting latitude/longitude coordinates, so I can take a guess that they’re
using one of the Google Maps APIs.

Since the map is realtime and rendered asynchronously I believe that they’re
using the [Google Maps JavaScript API](https://developers.google.com/maps/documentation/javascript/) - currently v3.  Now that we know what
service they’re using to plot the coordinates, the next question we must ask is - Where is the app retrieving the bus data containing the longitude/latitudes
of the buses from?  The most common way for apps to communicate with each other
is through an API, or [Application Programming Interface](http://www.makeuseof.com/tag/api-good-technology-explained/), and this is my first
guess as to where the data used in the map is coming from.  So, the next thing
to do is to see if we can locate any of the endpoints of this possible API so
we can consume the data in its raw form.

A good place to start is by opening your developer tools in your browser of
choice - I happen to be using Chrome and can do so with *option-command-i*.  The
Network tab shows all of the HTTP requests and responses to and from various
resources.

![live-view](/assets/hacking-your-environment/network-requests.png)

Each column represents a response.  We are looking for a column displaying a
GET request to some endpoint that returned a 200 status meaning successful.
Scrolling down through the columns we come across this *livebus.json.php* file,
which stands out to me for a few reasons.

![live-view](/assets/hacking-your-environment/livebus-json.png)

![live-view](/assets/hacking-your-environment/request-url.png)

1. It is named livebus - This filename tells us that it most likely has to do
with a bus and possibly live data about said bus (sounds like something we
might be interested in)

2.  It is saved with a .json file extension.  JSON stands for JavaScript Object
Notation and is one of the two most common data formats used to interact with
APIs - the other being XML.

3. JSONP - if you look in the left column you will see the Request URL ends
with *?callback=jQuery110204*… This actually has a lot of significance.  It shows
that the client is using JSONP to request the data, a technique used to request
data from a server in a different domain, which is actually prohibited by most
modern browsers due to the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy).

4. IP Address: The Remote Address is *128.197.26.3*.  Navigating to this in a
web browser shows it is hosted on one of BU’s servers.


![live-view](/assets/hacking-your-environment/ip-address.png)

Taking all of these points into account, it looks like we might have just found
the elusive API.  The only option left is to request this resource, which can
be done by navigating to the Request URL in your browser (essentially sending a
GET request).

You’ll notice below that we are served a page full of JSON and if you look
closely at the first attribute - “title” you’ll see that its value is “Bu Bus
Positions” - SUCCESS!

![live-view](/assets/hacking-your-environment/json-response-1.png)

While this big blob of JSON is a lot of fun to stare at, we are developers, and
as developers we work with code.  Let’s highlight this mess and save it to a
JavaScript variable so we can start diving into what we’re working with.

![live-view](/assets/hacking-your-environment/json-response-2.png)

Opening our Chrome Developer Tools and saving the JSON to a variable named
results we can begin parsing the response.

![live-view](/assets/hacking-your-environment/json-response-3.png)

The Dev Tools give us a much cleaner interface for working with this data.  We
can see a couple of attributes - *title*, *service*, *ResultSet*, etc.

![live-view](/assets/hacking-your-environment/result-1.png)

Expanding *ResultSet*  and then *Result* we notice 7 objects, numbered 0 - 6, which
turn out to be the 7 BU Shuttle Buses currently in operation.


![live-view](/assets/hacking-your-environment/result-2.png)

Diving into the first one you’ll notice we can get a lot of good data from each
bus including a *latitude* (lat), *longitude* (lng), *speed*, *callname*,
*general_heading*, etc.


![live-view](/assets/hacking-your-environment/result-3.png)

To get to the array of buses we called *ResultSet* followed by *Result*.  Let’s
save this to a variable, *buses*.  Now, to access the array of buses we can call
`buses` instead of having to call `results.ResultSet.Result`.  This helps both  now
and down the road when we forget what `results.ResultSet.Result` actually
returns.

![live-view](/assets/hacking-your-environment/result-set-1.png)

![live-view](/assets/hacking-your-environment/result-set-2.png)

We can now access each bus via its respective index. So for the first bus we
can use:

![live-view](/assets/hacking-your-environment/bus-1.png)

Looks like a prime time to use a simple for-loop to iterate through each
element (bus) in the array and extract some data about each.

![live-view](/assets/hacking-your-environment/bus-loop.png)

Programming is all about minimizing complexity.  Let’s make this program a
little more readable/maintainable and extract some variables.  We’ll get each
bus’s respective latitude, longitude, and speed and save these to variables
we’ll call *busLat* and *busLng*, and *busSpeed*.  By concatenating some strings with
these variables we can piece together the *locationAndSpeed* of each bus and log
this to the console.  Executing this loop provides the following output,
exactly what we need to start plotting the coordinates on our own map.

![live-view](/assets/hacking-your-environment/bus-loop-code.png)

![live-view](/assets/hacking-your-environment/bus-loop.png)

Software projects are best executed in multiple steps to limit complexity. This
is just the first step in a longer process of piecing together a fully
functioning application.  Every day we are presented with opportunities to
improve existing platforms and I challenge you, as the reader - the developer,
to go out and seize these opportunities.  Don’t be scared to build some shit,
and sure as hell don’t be scared to break some shit.     Stay tuned for part II
and feel free to reach out with any questions/comments:

@askwheeler

[http://github.com/alexwheeler](http://github.com/alexwheeler)

[http://github.com/alexwheeler/BuBus](http://github.com/alexwheeler/BuBus)
