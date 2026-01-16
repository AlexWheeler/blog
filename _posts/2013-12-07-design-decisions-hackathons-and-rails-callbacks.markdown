![yhakcs](/assets/yhacks.png)

The thought of spending a full twenty-four hours writing code, passing on a good nights sleep, and surviving off of countless cups of coffee may not seem like the most ideal way to spend a weekend in college, but just like anything in life *you can’t knock it until you try it*.  If you have never experienced a hackathon you sure have been missing out on an epic experience.  While these events do offer prizes to teams who successfully build the best project, hackathons are about much more than the prize money.  They are about learning something new, making new friends, and of course scoring backpacks full of free swag from sponsors.  I recently attended Y-Hack, Yale’s first hackathon, and as I expected it turned out to be an awesome time.  Besides walking away with a finished project, I made some awesome friends, learned some new concepts, and witnessed a rap battle at 2 am.  While I could spend hours discussing my thoughts on hackathons that isn’t the goal of this post.  Instead I would like to dedicate the rest of this post to go over a few of the design decisions I found myself having to make in the process of building my application, using Rails’s Callbacks, and some lessons learned.

I had been wanting to build a surfing app for quite some time and figured this was the perfect opportunity make it happen…in twenty-four hours.

My plan was to make the application as simple and seamless as possible for the user.  The first thing I do when planning out a programming project is to map out the flow of the app.  Mine went something like this:

user navigates to home page and creates an account —> redirected to page to chose location —> redirected to success page.

Simple enough.

With only twenty-four hours to submit your app at most hackathons you need make sure to keep things as simple as possible, while still making for an awesome user experience.  One of my biggest design decisions was to decide how I wanted to send the surf report via text message to the user.  I could have used Twilio, an SF based cloud company with an awesome API for integrating SMS with your application, however I chose not to for three reasons:

Why use an external service when rails can handle this functionality and I am trying to keep things simple?  I am beginning a new project dealing with emergency response that will utilize SMS, but can’t rely on external services during times such as hurricane, floods, etc -  this was a great opportunity to figure out how to handle sending texts with rails.  Lastly, Twilio wasn’t a sponsor and therefore was not offering any prizes for integrating their API.  Do you remember your earlier cell phone days? Or would we rather forget about those dark days? Did you ever happen to send a text message to an email address?  Well I did, and it worked, therefore I figured I could simply reverse this process and send an email to a cell phone.  After some quality research (simple googling) I confirmed my assumption and learned that all you need to do is append the proper ending based upon the carrier as seen below.  Since Rails does indeed ship with a mailer it looks like we’re all set to send some surf reports.

![phone-extensions](/assets/phone-ext.png)

Now, all I had to do was collect a user’s cellphone number and carrier and append the correct ending.  The first part was simply accomplished by adding a drop down to select one of the three most common carriers on my home page.

![smsurf](/assets/smsurf.png)

Okay, so we have a phone number and carrier.  Now we need some logic to tell Rails which ending it should append based upon the chosen carrier. The best tool for this job turned out to be a Callback.  Rails’s API defines Callbacks’ as *hooks into the life cycle of an Active Record object that allow you to trigger logic before or after an alteration of the object state*. In other words callbacks allow us to trigger some sort of logic before or after we do something to an object - in our case before an object is validated, and thus saved into our database.

My User model has four attributes: email, password, phone_number, carrier.

I defined three callbacks for the three different carrier options in my user model as so:


```ruby
# user.rb

def append_verison_ending
  self.phone_number = self.phone_number + "@vtext.com"
end

def append_att_ending
  self.phone_number = self.phone_number + "@txt.att.net"
end

def append_sprint_ending
  self.phone_number = self.phone_number + "@messaging@sprintpcs.com"
end
```

These methods take the objects phone_number attribute and simply concatenate the ending and the phone_number - both strings objects.

Next, we must tell our application when to append each ending based on the given conditions, so we make three more methods:

```ruby
# user.rb

def carrier_is_verizon?
  self.carrier == "verizon"
end

def carrier_is_att?
  self.carrier == "att"
end

def carrier_is_sprint?
  self.carrier == "sprint"
end
```

All these methods do is check the carrier the object’s carrier attribute and whichever one matches will return true.

Now by combining these methods we are able to  conditionally implement the callbacks:

```ruby
# user.rb

before_validation :append_verizon_ending, :if => :carrier_is_verizon?
before_validation :append_att_ending, :if => :carrier_is_att?
before_validation :append_sprint_ending, :if => :carrier_is_sprint?
```

And we are done! Now when a user record saves, the proper ending will be appended allowing us to send a text message to the user via Rails’ built in mailer.
