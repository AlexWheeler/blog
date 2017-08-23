---
layout: post
title:  "Passing Javascript Variables to Rails Controller"
date:   2014-01-26 09:36:14 -0400
categories: jekyll update
---

*“To the man who only has a hammer, everything he encounters begins to look like a nail.”* - Abraham Harold Maslow

Ruby is an amazing programming language.  It ships with beautiful syntax, an extensive standard library, and more open source libraries  (via gems) than we could ever know what to do with.  However, every job requires different tools, and while there are almost always multiple ways to address a problem, I firmly believe you should make your life easier and pick the right tool for the job at hand. Ruby works great on the server side of most web applications, however when it comes to client-side programming there is no doubt that JavaScript reigns supreme.  Theres a reason it is often referred to as the language of the browser.  An understanding of JavaScript is a must for any modern day web developer and with the emergence of many awesome [HTML5 APIs](http://www.creativebloq.com/html5/developer-s-guide-html5-apis-1122923) you would be missing out on a lot of fun if you didn’t have at least a basic understanding of it.

JavaScript runs on the client-side, while Ruby runs on the server-side of Rails projects.  Without going into too much detail, the internet is quite simple.  You type a URL into an address bar and see a website.  But what is actually happening? Well, a client, such as yourself, types in this URL and hits enter.  This sends an HTTP GET request to a server asking for the resource which is stored at that specific location.  Remember, URL simply stands for *universal resource locator*. In other words, a URL can be compared to the street address of some information that you want.  If your computer has provided the correct information about itself then the server answers with an HTTP response containing the stuff that you want, organizes the HTML files into a tree known as the DOM, at which point you are able to see the website how you’re used to seeing websites, and then runs the necessary JavaScript files.  This JavaScript then manipulates the content that has been delivered by the server - hence the name client side scripting.  Data getting sent to a server is quite similar.  To break it down very simply let’s just say that data is sent to a server via an HTTP POST request, The server receives the request and Ruby manipulates that data. If it receives a GET request for that data, the server proceeds to send it back to the browser (you) at which point JavaScript takes its turn again manipulating the returned content.

Most of the time you will be using JavaScript to manipulate content in the browser that has been returned from a server - and there will be times when you will want send some data back to the server for Ruby to work on.  How would you go about doing this?  JavaScript and Ruby are two different programming languages.  All of the code in between erb tags `<%= %>` is executed on the server, while any code between script tags `<script></script>` gets executed on the client. In other words these languages have no idea each other exists.  The easiest way to get JavaScript data back to the server is how most new data gets there - a form.

I came across this very problem recently in a project I’m working on. The premise of the app is that it gets a user’s current Latitude and Longitude via the HTML5 Geolocation API, makes a call to the Google Maps API, which returns a link to an image of that user’s Lat and Long marked on a map, followed by an API call to the Bit.ly API to retrieve a shortened URL of this link, all because finally I must POST this data to the Twilio API via a create action in one of my controllers to send a text containing this data to another user’s phone number.  The problem is that all of this happens on the client side via JavaScript and I must get it to my controller to POST to Twilio, save as a record in the DB and possibly call on some more external services in the feature (its always best to keep your software as flexible as possible for future iterations).

I knew that if I could somehow get the data into a form I could submit it to the controller.  Since JavaScript allows us to manipulate just about anything on a web page, then this problem should be no challenge for such a powerful programming language.  *Don’t worry If you’re not even a little bit familiar with JavaScript/jQuery or the DOM, the following will only be a high level overview of my code.*

The app is quite simple.  A user that wants to request a friend’s location visits a home page.  He or she fills in their phone number and a phone number for the friend who’s location they want.

image

 A hyperlink is sent as a text message to the friend’s phone with a query string containing the original senders phone number set to the phonenumber parameter (my next post will cover how we solved data persistence in this app). When the friend clicks the link, a url of my app is opened, containing some JS code that executes upon page load. This JavaScript doesn’t only get the user’s lat/long, but also is the key to passing itself to my controller for further processing.  So, how does this work?

I have a form routed to the create action of my twilio_messages_controller:

*twilio_messages/new_html_erb*

```erb
<%= form_for @twilio_message do |f| %>
<%=   f.hidden_field :to, valu:e params[:phonenumber] %><br>
<%=   f.hidden_field :from, value: "+19498607430" %><br>
<%=   f.hidden_field :body, value: "" %>
<%=   f.submit "Send ETA!", class: "submit enabled" %>
<% end %>
```
*twilio_messages_controller.rb*

```ruby
def create
  account_sid = ENV["TWILIO_ACCOUNT_SID"]
  auth_token = ENV["TWILIO_AUTH_TOKEN"]
  @client = Twilio::REST::Client.new(account_sid, auth_token)
  @client.account.sms.messages.create(twilio_message_params)

  @twilio_message = TwilioMessage.new(twilio_message_params)

  if @twilio_message.save
    flash[:phone] = params[:twilio_message][:to]
    redirect_to twilio_message_success_path
  else
    flash[:phone] = params[:twilio_message][:to]
    redirect_to twilio_message_failure_path
  end
end

private

def twilio_message_params
  params.require(:twilio_message).permit(:to, :from, :body)
end
```
It has two hidden fields, which aren’t visible to the user.  The form, seen below, actually just looks like a button, however it is indeed a form.

![Geofun form](/assets/geofun.png){: .center-image }

The *To* value is set by accessing the query string the original text message body appended on to the link that the friend opens.  The *From* value is always going to be my Twilio Number so this value is hardcoded (you don’t have the account_sid or auth_token so I’m not too worried).  They body is the text message body that will be sent.  In my case, I want it to be a map with the user’s location marked with a pin.  Remember how I stated how when the user opens the page it executes some JS code?  Well we’re now in a position to tie everything together and explain this in greater detail.  When the friend opens the text message link JavaScript code is executed that retrieves the device’s latitude and longitude via the HTML5 geolocation device API, sends the lat/long to Google Maps API, which returns a link to a google map with the user’s location marked on it. Then I take this url and make an AJAX request to the Bit.ly API to shorten the url since Twilio only allows text message bodies < 160 characters and the url returned by Google Maps is very long.  Now I have the data stored in a JS variable and just need to get it in the form somehow - easy.

Browsers provide us with Javascript functions called event handlers, which allow a developer to trigger a certain function whenever an event occurs on a target element.  In other words you attach an event handler to a certian element and when a user does something to this element, something happens.   This could translate to a user clicking on a link and an alert box popping up, or a user scrolling over an element on a page and all of the images changing to different images.  You’ve probably had this happen to you countless number of times - and now you know you have JavaScript to thank for it.

One of these event handlers is onready (jQuery calls it *ready()* ), which fires a given function when all of the content on a page is done loading and it is ready to be manipulated by JavaScript.

```javascript
$(document).onready(function() {
  getMap();
});
```
Put most basically, I wrapped all of my API calls in a ready( ) event handler so that when the friend loaded the page they all fired and the result was a Bit.ly shortened link containing my reference to the Google Maps API. I then took this variable and used some more of the power of Javascript to set that link as the value of the Body input. (*note: the data.data.url parameter is just parsing the JSON response from Bit.ly*)

```javascript
$.getJSON(shorturl, function(data) {
  $("#twilio_message_body").val(data.data.url);
});
```

The form is essentially completed by this JS and the friend only has to click the Send ETA button. This sends a POST request to the create action of my controller, with the request body being the form inputs, which are accessable like any other parameters in a controller:

```ruby
params[:text_field_name].
```

i.e. `ruby params[:to], params[:from], params[:body]`.

If you’re using Rails4 don’t forget to whitelist these params in the corresponding controller.

```ruby
def twilio_message_params
  params.require(:twilio_message).permit(:to, :from, :body)
end
```
And thus the data I needed, which was stored in JavaScript variables has successfully been sent to my controller, saved to my database, and sent as an sms via Twilio:

*twilio_messages_controller.rb#create*

```ruby
@client = Twilio::REST::Client.new(account_sid, auth_token)
@client.account.sms.messages.create(twilio_message_params)
```

Feel free to reach out if you have any questions or want to go more in depth about certain topics covered.

[@askwheeler](http://twitter.com/askwheeler)

Find the GitHub repo [here](https://github.com/AlexWheeler/eta)

Special thanks to great friends [@NateChaseH](https://twitter.com/NateChaseH) and [@netspencer](https://twitter.com/netspencer) working on this app with me, making it beautiful, and continuously inspiring me.

Also check out my great friend [John Capecelatro’s blog](blog.johncapecelatro.com).  He is the one friend I know I can count on to be up on Saturday mornings ready to get together and hack - even after a long night out on the city.
