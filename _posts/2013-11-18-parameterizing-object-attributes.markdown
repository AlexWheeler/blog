---
layout: post
title: "Parameterizing Object Attributes"
date: 2013-11-18 09:36:14 -0400
categories: ruby rails
---

I found that querying by category worked great.  The URL for all of the moving
jobs was simply */moving*.  But what about the URL for a category that had
multiple words separated by spaces?  Would the Home Services be found at */Home*
Services.  Unfortunately, the answer is no.  It turns out that when a string
gets encoded as a URL all of the spaces get encoded as %20.  If you’re curious
you can read here about why this occurs.
[https://tools.ietf.org/html/rfc3986#page-12](https://tools.ietf.org/html/rfc3986#page-12),

![param query main](/assets/param-query-main.png)

If you don’t have the time or could care less than just know that spaces in
strings get encoded as *%20* in a URL.  From an SEO standpoint this is not good
and is actually quite confusing to a user.  Since we want our app to be as user
friendly as possibly a user should be confident that when they want to see a
listing of the Home Services jobs they can be found at a predictable location.
*/Home%20Services* is far from predictable.  After doing some research I found
that the most common way to deal with spaces is to replace them with a simple
“-”.  This is much more SEO and user friendly.  It is much easier for a user to
remember that any category with a space need simply to have the space replaced
with a “-” rather than %20.

Solution: I knew that I wanted to replace the spaces in the URL with “-”, but I
also knew that I still wanted the select box for choosing a category in the
form to have spaces.

so that it looked like this:

*_form.html.erb*

![good form](/assets/form-good.png)

and not this:

*_form.html.erb*

![bad form](/assets/form-bad.png)

This meant that I would need to replace the spaces after the form is submitted.
Well, the simplest way to edit an attribute after a form is submitted but
before the record is saved is with a callback.

First, I found out that active record has an actual method `parameterize`, which
takes a string object, sets it to lower case and replaces any spaces with “-”.
Perfect, this is exactly what I was looking for.  I defined a private method
called #parameterize_category in the Job model.  It simply calls Active
Record’s #parameterize on a job object’s category attribute.

*job.rb*

```ruby
private

def parameterize_category
  self.category = self.category.parameterize
end
```

The next step was to make this a callback by telling rails that before it saved
a job object to parameterize its category:

Job.rb

```ruby
before_save :parameterize_category
```

Great now we have predictable, SEO friendly URLs as can be seen below.  But it
looks like we have one more problem.

![param title bad](/assets/param-title-bad.png)

Notice that since our header tag is dynamic as `params[:category]` it is
displaying the parameterized category.  Once again this looks like crap and
while it might not be the end of the world I think it would look a lot better
if it said Child Care and not child-care.  This means we need to find a way to
`unparameterize` this, but only in the view since we still need the parameterized
string in the URL.  If only Rails provided us with something to deal with
methods we only want used in our views.  Lucky for us, it does, which is
exactly what helper methods are for.

In jobs_helper.rb I defined a method `titleize`:

*jobs_helper.rb*

```ruby
def titleize(title)
  title.gsub("-"," ").titleize
end
```

`titleize` takes one paremter - a title.  It uses Ruby’s `gsub`, which takes two
parameters. The first being a character you wish to replace and the second
being what you would like to replace this character with.  In our case we want
to replace any “-” with a space.  After this it calls Active Record’s #titleize
on this object, which capitalizes the first letter of each word in the string.

In my index view I replaced `params[:category]` with:

```erb
<h2><%= titleize(params[:category]) %></h2>
```

which results in:

![param title good](/assets/param-title-good.png)

And there we go.  We have much prettier URL’s and titles.


