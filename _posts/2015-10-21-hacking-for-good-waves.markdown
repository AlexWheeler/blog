*Note: Surfline has been contacted and is working on patching the issue. They
gave permission to post this and hooked up a year premium membership. Very
metal  Surfline \m/.*

Every surfer knows a Rex. He’s the guy who always seems to be in the right spot, at the right time, every time. Rex actually understands all these terms us surfers love to throw around while sizing up a 5-7ft@21 seconds WNW swell cranking down from the Gulf of Alaska at 280 degrees. While you and the rest of the boys froth over how hard you’re going to score at the local beach break in the morning, Rex is already running the numbers in his head, knowing this time tomorrow he’ll be scoring head-high, empty barrels, while you duke it out with the rest of the weekend warriors in two foot slop. He was right, you were wrong. God damnet.

*Rex - Baja Norte, Mexico 2014 (yes we scored)*

![rex](/assets/hacking-for-good-waves/rex.png)

While you may know a Rex, you may or may not know anything about the world wide web or how we ended up where we are today.

In the early days of the web there really wasn’t a way to programmatically make changes to a web document after a request succeeded. Any modifications to a document had to be made as it dynamically rendered on the server. Then during the mid 1990’s, in the midst of an era that would later come to be known as the browser wars, emerged the amazing language that we all know as JavaScript.  JavaScript opened the doors to client-side scripting, giving application developers the ability to run code in the context of a web document within the browser after it returned from the server! It also opened the door to countless security vulnerabilities and gave people like me an entire playground to explore to our hearts content.

While I certainly believe I have one of the coolest jobs in the world, spending most of my days trying to find more efficient ways to tell computers what to do, I have also had the amazing opportunity of pissing away the majority of my youth enjoying one of life’s greatest gifts - surfing. Even since moving to NYC a few months ago I’ve made it a point to get in the water nearly every weekend there are waves. At first glance these may seem like two opposing worlds, but long gone are the days of Bruce Brown’s [Endless Summer](https://en.wikipedia.org/wiki/The_Endless_Summer). We’re in the 21st century, meaning surf forecasting has not only entered the digital age, its thriving. While our friend Rex can easily decipher the complex global swell models and forecasting tools built on top of publicly available data from the likes of the National Data Buoy Center, most of us just want to know how the waves are looking right now. From there we can make educated guesses as to how they’ll look in an hour or so.

*Long Beach, NY summer 2015*

![rex](/assets/hacking-for-good-waves/long_beach.png)

Any surfer who’s actively surfed in the past decade is no doubt familiar with [Surfline's](www.surfline.com) camera services that allow anyone with an internet connection to check the waves at thousands of spots around the world in real time. As with all awesome things in life there’s a catch. Unless you’ve shelled out the money for a premium account the stream pauses, displays an advertisement, and forces a refresh every 30 seconds. Its an accepted fact by now that this will continually happen just before the set you’ve been waiting for is seen rolling in from the horizon. By the time the stream refreshes said set is long gone. If only we had a few more minutes without the refresh we could make a sound decision whether to head for the beach or try our luck tomorrow. Turns out today I don’t even need to check the cams to see what the swell is doing.  Looking out the window tells me the weather is crap and winds are picking up.  There couldn’t be a better day to grab some wine, get comfy on the couch, and ~~watch netflix~~ read source code!

We already know two very important pieces of information.

1. JavaScript allows developers to manipulate client-side content

2. Surfline’s cameras pause after 30 seconds to show an ad

Its not too difficult to put the two together and conclude that Surfline is most likely using JavaScript to trigger an event that pauses he video stream and displays the ad. Since your web browser downloads all of the requested resources from the server we can go ahead and dive into some of Surfline’s code.

The first step is to locate the scripts. This is simple in most modern, evergreen browsers - my preference being Chrome. Simply open the developer console with *Cmd + Shift + i* and navigate to the sources tab. Here we can find all of the downloaded JavaScript files.

![sources1](/assets/hacking-for-good-waves/sources1.png)

Most of them will be from third party applications they’re running such as ad clients, analytics tools, etc., but at the top level of the directories to the left we quickly find the code we’re looking for in a ColdFusion Markup File *report-c.cfm*, nested under surfline.com/surfdata.

Now the fun begins. We want to find any events trigged on a camera. So how about we start by searching for any code containing the word *camera*?

![sources2](/assets/hacking-for-good-waves/sources2.png)

Toggling through the 134 results we soon come across our first interesting statement.

Clearly this is initializing a variable called `cameraCurrentTime` to 30000 milliseconds - a very common time unit used in JavaScript programming. Some elementary math tells us 30000 milliseconds is equal to 30 seconds - the same time frame the camera runs for before requiring a refresh!

![sources3](/assets/hacking-for-good-waves/sources3.png)

Digging a bit deeper we find the culprit - `updateTimer()`. Every time this function is called it first checks if `cameraCurrentTime` is equal to 0. If it is equal to 0 it ends the camera stream, otherwise it decrements `cameraCurrentTime` by 1000 milliseconds or 1 second.

Heading another level deeper we discover a magical line of code on line 5268 that informs us of two things.

![sources4](/assets/hacking-for-good-waves/sources4.png)

There is a variable `userType` defined somwhere in this file that can be one of many different values, some of which are:

* premium

* vip

* vip_adfree

* vip-adfree

I’m just going to take a wild guess here and say that these values are probably pretty important.

Whoever fulfilled Gavin’s request really didn’t think this one through.

The next step is to find out where `userInfo` is initalized. This could take a long time given this is a 10,666 line file, but since we’re technologists we’ll let our machines do the work. A quick cmd-f search for ‘userType = ’ (notice whitespace to avoid matching userType == boolean checks) reveals the keys to scoring better waves for free. Dissecting these eleven lines of JavaScript should tell us exactly how to set a user’s privilege level.


![sources5](/assets/hacking-for-good-waves/sources5.png)

Line 4935 informs us user information is stored in a cookie `'USERINFO'`. Cookies are just small text files stored on disk that websites often use to keep information about a user.

![sources6](/assets/hacking-for-good-waves/sources6.png)

Line 4938 parses `userInfo` as a JSON string, transforming it from a string to a basic JavaScript object, an interface for data storage that’s much easier to work with than a standard string.

![sources7](/assets/hacking-for-good-waves/sources7.png)

Line 4939 accesses the value we care about. It begins by asking `userInfoParsed` for its user attribute, which returns an array. This array stores an object at its first index (arrays in JavaScript are 0 indexed). Asking this object for its `usType` returns the current user’s privilege level.

As a free member we won’t have a USERINFO cookie stored, however that doesn’t mean we can’t store one before Surfline can check for it. As far as the JavaScript is concerned, Surfline set this cookie and because of this we can trick the browser by simply reversing the operations, setting the cookie ourselves, and reloading the page, in turn re-executing the script - this time with an escalated privilege level.

**reversing the steps**

1. Build data structure

    ![sources8](/assets/hacking-for-good-waves/sources8.png)

2. Transform JSON object to JSON string via `JSON.stringify()` (opposite of JSON.parse)

    ![sources9](/assets/hacking-for-good-waves/sources9.png)

3. Set cookie USERINFO to this value

    ![sources10](/assets/hacking-for-good-waves/sources10.png)

4. Confirm it works :)

    ![sources11](/assets/hacking-for-good-waves/sources11.png)


Now when the browser executes this script it will find the `'USERINFO'` cookie, parse it, and grant premium access.

![still-watching](/assets/hacking-for-good-waves/still-watching.png)

To this day Surfline remains, hands down, one of my favorite technology companies. They’ve no doubt played an enormous role in helping my friends, as well as the rest of the surf world, score epic surf sessions year after year.  The ethical action for any software professional to take upon discovering a security issue is to follow the [responsible disclosure](https://en.wikipedia.org/wiki/Responsible_disclosure) process by notifying the vendor and allowing them a reasonable amount of time to patch it before notifying the public. (Unless its Fox News - then please feel free to burn it to the ground). But really, if you find an issue with a vendor’s site that you respect  report it! After disclosing the vulnerabilty and providing a proof of concept the epic team at Surfline has addressed the issue, upgraded my account to premium for the next year, as well as given permission to publish the post.  Rock on \m/

![email](/assets/hacking-for-good-waves/email.png)
