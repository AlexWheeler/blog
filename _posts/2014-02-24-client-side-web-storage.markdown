Improving as a developer undoubtedly takes a whole lot of time, a great deal of discipline and what might seem like an even greater amount of frustration.  This IS hard stuff - then again I don’t remember anyone saying it would be easy?  Some concepts can take hours if not days or weeks of studying to fully wrap your head around, while others can be comprehended in a matter of minutes.  Technology is advancing at an insanely rapid pace and as developers we must constantly keep one eye on past/existing technologies, while simultaneously directing the other towards the future.  It is strange to try to think about where you learned everything you have come to known about the world around you.  There are some things that are very easy to remember.  Take for example learning the alphabet in kindergarden or about bell curves in high school statistics. Others are a bit harder to remember as we seem to have gradually picked them up over time.  I clearly remember a friend explaining to me what a string is, but do I remember how I learned the difference between strict equality (===) and normal equality (==) in JavaScript?  Perhaps, someone sat down and explained it to me at a local hack night or quite possibly I heard it on a podcast somewhere.  While I don’t think it really matters where we learned something (as long as that information is correct) it is an interesting subject to think about.  Anyways this is my technical blog so maybe we should discuss some technical stuff.  The reason I brought up the subject of thinking about where we learn things is because I’ve noticed there are certain concepts/tools we pick up that seem to completely change our perspective on problem solving.  For example, learning some obscure UNIX command might (arguably) not have a large impact on my programming, while some other small bit of knowledge will.

Recently I’ve found myself playing with a lot of the HTML5 JavaScript APIs as seen in my last post.  I stumbled across the Web storage API, that actually technically isn’t included in the HTML5 specification anymore due to some dumb political reasons, but we’ll pretend that it is since it resembles most of the other HTML5 APIs.  This proved to be one of those small bits of knowledge that completely opened up an entire new world of approaching problems, mostly related to data persistence.

Web Storage allows web developers to store key-value pairs of information about a user on the client side without ever having to share this data with the server.  It sounds a lot like cookies, right?  True, they are similar in that they are both used to store information about a user on the user’s machine, however they do differ slightly giving each an advantage in its own right in varying situations.  You may or may not know what a cookie is, or quite possibly you just remember some friend when you were younger telling you to clear your cookies so your parents can’t see what websites you’ve been visiting (oh, the good ‘ol days).  Cookies are just text-files that web servers store on a user’s machine that can later be retrieved.  For example, when you want to check your Facebook you visit www.facebook.com by typing this into your address bar in the browser and hitting enter.  Your browser sends an HTTP request to Facebook’s servers asking for its homepage and also takes a look on your computer for any cookie files that Facebook has set and sends those along in the request headers.  In case you ever wondered how Facebook and similar sites are able to remember your username and password and already have them filled in by the time you visit the site, this is done with cookies.

Like any technology there are a few problems with cookies.  Cookies are sent in the request headers of every single HTTP request to the server, which can consume a lot of network bandwidth, while making for slower request times.  Another problem is that cookies are quite small and can only store 4KB of data.

Cookies have been around for the majority of the web’s existence.  As websites transitioned into more robust web apps we found ourselves seeing the benefits of storing information on the client side, and because of this newer methods have been introduced to handle this task.  One of these is the Web Storage API,which is what I would like to discuss.

Web Storage allows websites to store up to 5MB of key-value pairs of information on the client-side, although it is recommended to only expect to be able to use about 2.5MB per website since all browsers have different limits.  A great advantage of Web storage is that it never communicates with the server so your site won’t find itself passing unnecessary information in every request.  And of course you can always send Web storage data to the server with a simple AJAX request if you need to.  While this is all super cool the absolutely best part about the Web storage API is how simple it is to implement.  There are two basic objects that you will be working with.

1. localStorage

2. sessionStorage

*localStorage* is persistent across all pages that are in the same domain and will be stored until explicitly removed.  The user can navigate to a new tab, close the current tab, open a new window and even quit out of their browser and the data will still persist.

*sessionStorage* is only available on the tab that it is created on and will be removed once the tab is closed.

Those are the key differences between the two objects and for the rest of the post we can focus on localStorage.

Getting started working with localStorage is as easy as manipulating the localStorage object.  There are four main methods that you will care about.

1. `setItem(key,value)`

2. `getItem(key)`

3. `removeItem(key)`

4. `clear()`

`setItem` takes two parameters, key and value, and creates a key and a corresponding value to be stored.  For example:

```javascript
localStorage.setItem('name’,'Alex’)
```
will set name => 'Alex’

You can then retrieve this value with getItem as seen below:

```javascript
localStorage.getItem("name");
```
`=> “Alex”`

`localStorage.clear()` will clear the localStorage object of any information

`removeItem()` removes the key value pair for whatever key you pass as a
parameter.

And just when you thought it couldn’t get any easier it does.  You can set and retrieve these key-value pairs just as you would any JavaScript property - since in truth they really just are properties of an object.

```javascript
localStorage.name = “Alex”;
```

```javascript
localStorage.name;
```
`=> "Alex"`

That really is all there is to the LocalStorage API.  Below I will post a very basic idea for when you would use it and how I went about implementing it with some jQuery and Javascript.

In my previous blog post  I mentioned a recent app some friends and I built that requires the user to type in their phone number and a friend’s phone number and click enter.  From a user point-of-view we wanted the app to be as streamlined as possible  - no download necessary, any OS, fill in as little information as required, etc.  Since the user’s phone number will always be the number from the phone they are accessing the app from, there is no reason for the user to have to retype it every single time they would like to use the app.  What if the app remembered their phone number and all he or she had to do was type in the friend’s phone number?  This would cut the amount of typing the user has to do in half!  Well, with localStorage this is possible.

![geofun 1](/assets/client-side-web-storage/geofun-form-1.png)

Above is the initial form.  Why should a user have to type in his or her phone number every single time?  Wouldn’t it be easier if they entered a phone number once (for example: 999-999-9999) and then every time they came back to the site it would already be entered?

![geofun 2](/assets/client-side-web-storage/geofun-form-2.png)

We can solve this by saving the user’s phone number into localStorage using jQuery/JavaScript and then retrieving this information every time the user visits.

```javascript
function storePhoneNumber() {
  if (!localStorage.phoneNumber) {
    $("#button").click(function() {
      var phoneNumber = $("#twilio_request_message_from").val();
      localStorage.phoneNumber = phoneNumber;
    })
  }
}
```

I define a function storePhoneNumber.  Line 27 checks if the localStorage object contains a phone number property.  if this returns false then an event handler is placed on the form submit button so that when the user submits the form localStorage.phoneNumber is set to whatever the user entered as his or her phone number.

```javascript
function setPhoneNumber() {
	if(localStorage.phoneNumber) {
    $("#twilio_request_message_from").val(localStorage.phoneNumber);
  }
}
```

I define another function setPhoneNumber, which checks if localStorage has a phoneNumber property and if it does it populates the input field with this phone number.  A simple refactoring could combine these two functions in a conditional and thus you have an easy-to-read function that does exactly what we want.

```javascript
function handlePhoneNumber() {
  if(localStorage.phoneNumber) {
    getPhoneNumber();
  }
  else {
    setPhoneNumber();
  }
}

function setPhoneNumber() {
  $("#button").click(function() {
    var phoneNumber = $("#twilio_request_message_from").val();
    localStorage.phoneNumber = phoneNumber;
  })
}

function getPhoneNumber() {
  $("#twilio_request_message_from").val(localStorage.phoneNumber);
}
```

Hope you enjoyed and feel free to get in touch with any questions/comments!

[@askwheeler](https://twitter.com/askwheeler)

[github.com/alexwheeler](https://github.com/alexwheeler)
