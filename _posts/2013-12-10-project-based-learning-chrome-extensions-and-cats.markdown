---
layout: post
title:  "Project Based Learning, Chrome Extensions, and Cats"
date:   2013-12-10 09:36:14 -0400
categories: Javascript Chrome Extension
---

Everyone has their unique way of learning.  Understanding and embracing this idea is one of the most
important steps towards accelerating one’s learning.  There are three main types of learners:
auditory, visual, and kinesthetic.  Auditory learners prefer listening to concepts being explained,
visual learners learn by looking at graphics, and kinesthetic learners learn by touch or *hands-on*
experience.  Some people only identify with one of the three types, while others can relate to a
combination of all three.  Take for example the petite Chinese girl that sits next to me in one of
my economics classes.  Every day she records the lecture on her iPhone so that at a later date she
is able to listen and watch the lectures again and then complete problem sets to truly nail the
concepts.  She obviously identifies with all three types.  She also enjoys heavy metal and walking
into class with her headphones on full blast jamming Slayer - the last thing you would guess from
the pint-sized, quiet international student, but she knows she enjoys it and that’s totally badass.
Anyways, the point is that everyone has a different way of learning and identifying how you learn
best is critical towards accelerating your learning.

I can confidently say that I am a kinesthetic learner - I learn best by doing.  I know that I better
understand concepts from my classes after completing a few problem sets. Knowing this I am able to
apply it to any sort of learning I am doing - especially programming.  My most recent learning has
involved mastering JavaScript.  JavaScript is an essential tool for developing modern web
applications.  This idea combined with the allure of the many JS frameworks gaining momentum are two
of the main reasons I’ve chosen this as my next language to learn .

Despite what you may have heard JavaScript is actually an extremely useful, powerful language,
especially when you take into account that Netscape only gave Brendan Eich about ten days to design
it. (side note: the history of JS is extremely interesting, check out some Douglass Crockfords
videos on the subject).  Combined with the DOM or Document Object Model, javascript allows us to
manipulate just about anything on a given page.  To put it shortly, when you, the client, send an
http GET request to a server, that server responds with the necessary files, and assembles the HTML
into a tree consisting or parent nodes and child nodes, which we, the developers, with a little bit
of javascript knowledge can access and manipulate.  With permission, chrome extensions can access
many things ranging from the current tab’s DOM to Facebook and twitter data.  Yes, the possibilities
are endless.

So, I knew I wanted to build a chrome extensions, while deepening my understanding of javascript.
As any other rational person would have done I decided to build an extension that would replace all
of the images on a given page with pictures of cats and money.

After setting up a few files that the chrome web store requires to be included in your app it was
time to begin building.

There are two main files to the app:

1.) background.js - simply tells chrome to run a file I named script.js when the user clicks the
extension’s icon.

```javascript
chrome.browserAction.onClicked.addListener(function(tab) {
  chrome.tabs.executeScript(null, {
    file: "script.js"
  });
});
```

2.) script.js - contains my JS code that manipulates all of the images.

```javascript
  var catImages = ["cat1.png", "cat2.png", cat3.png",
                   "cat4.png", "cat5.png", "cat6.png",
                   "cat7.png", "cat8.png",  "cat9.png",
                   "cat10.png"]

var numberOfImageNodes = document.getElementsByTagName("IMG").length;

function bringOnTheCats(numberOfImages) {
  for (x=0; x < numberOfImages; x++) {
    var randomNumber = Math.floor(Math.random() * 10);
    var image = document.getElementsByTagName("IMG")[x];
    image.src = chrome.extension.getURL(catImages[randomNumber]);
  }
}
```

Breaking down the problem at hand I figured I needed:

1.) a collection of cat images to replace the original images.

2.) a value for how many images the page contained since I can’t change the image source for an
undefined object.

3.) A loop to iterate through the images and change their source

Let’s break down my code:

Arrays are one of the most common data structures used in programming and for good reason, they are
indexed and able to be iterated through, therefore they seem to work really well with loops.

I begin on line 1 by making an array *catImages* that contains all of my cat image files.

```javascript
var catImages = ["cat1.png", "cat2.png", cat3.png",
                 "cat4.png", "cat5.png", "cat6.png",
								 "cat7.png", "cat8.png",  "cat9.png",
								 "cat10.png"]
```

Next, I create a variable called numberOfImageNodes, which simply counts how many images the page
contains.  To do this I call #getElementsByTagName(“IMG”) on document, which returns an array of all
of the elements with an image tag. Then I simply call length on this returned array, which tells me
how many values the array contains, translating to how many images the page has.

```javascript
var numberOfImageNodes = document.getElementsByTagName("IMG").length;
```

Now, comes time for the function that is going to tie all of these together.

I create a function called bringOnTheCats that takes one parameter - numberOfImages.

```javascript
function bringOnTheCats(numberOfImages) {
  for (x=0; x < numberOfImages; x++) {
    var randomNumber = Math.floor(Math.random() * 10);
    var image = document.getElementsByTagName("IMG")[x];
    image.src = chrome.extension.getURL(catImages[randomNumber]);
  }
}
```

*Note: this function doesn’t necessarily need this parameter and I could have left it out and
replaced numberOfImages on line 6 with numberOfImageNodes since I declared it as a global variable,
but I figured by passing a parameter my function would be more flexible for future iterations.*

Within this function on line 6 I create a loop that sets a variable x to 0, and states while x is
less than the number of images on the page, increase x by 1 after each iteration.

```javascript
for (x=0; x < numberOfImages; x++)
```

On line 8 I create a variable randomNumber, which on every iteration gives me a new random number
ranging from 0 - 10. Breaking this line down: Math.floor rounds down whatever number it is passed as
an argument. `Math.random()*10` returns a random number ranging from 0-10, however it is not always a
whole number, so we must round it down hence why we combine the two to make
`Math.floor(Math.random()*10)`.

```javascript
var randomNumber = Math.floor(Math.random() * 10)
```

Lines 9 & 10 are where we truly tie everything together and where the real magic happens.  As I
explained earlier #document.getElementsByTagName(“IMG”) returns an array containing all of the
images on the page. On every iteration of the loop, an image at index x of that array has its source
set to an image file at a random index (between 0-10) from the catImages array.

```javascript
var image = document.getElementsByTagName("IMG")[x];
image.src = chrome.extension.getURL(catImages[randomNumber]);
```

Line 13 calls the function bringOnTheCats and passes in the numberOfImageNodes and we are done.
Now, if a user downloads the extension, navigates to a url, and clicks on the icon, this function
will be called and cats will take over the user’s page.

```javascript
bringOnTHeCats(numberOfImageNodes);
```

Now that you know how it works, see it in action:

[Pussify](https://chrome.google.com/webstore/detail/pussify/kalnkogdemjhapdnhafhmciodhfbajlc?hl=en) in the chrome web store

Source code on [GitHub](https://github.com/AlexWheeler/cats_chrome_ext)
