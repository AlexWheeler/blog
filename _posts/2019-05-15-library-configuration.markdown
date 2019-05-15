---
layout: post
title: "Library Configuration"
date: 2019-05-15 09:36:14 -0400
categories: ruby
---
At some point in your open source project's lifetime users are going to want the ability to configure your software to suit their needs.  They'll want to add their own text to a dashboard. They'll need to disable the ability to take some action.  They'll want to rip out your [Taylor Swift video](https://github.com/jnunemaker/flipper/pull/384) from the UI.  FWIW I actually find the Taylor Swift embed pretty funny.

What if I told you there was a way we could all be happy?  And, no, the answer doesn't involve monkey patching. The easiest pattern I've come across involves introducing a `Configuration` class:

```ruby
class Configuration
  attr_accessor :title, :text

  def initialize
    @title = "Title Default"
    @text = "Text Default"
  end
end
```

Adding a getter to your library's base class to retrieve a `Configuration` instance:

```ruby
class Library
  def self.configuration
    @configuration ||= Configuration.new
  end
end
```
Note `@configuration` is memoized so the first time `configuration` is called the instance variable is set and any following calls will return this reference instead of initializing a new instance each time.


Finally providing a `configure` method that yields the configuration - making our instance *configurable*:

```ruby
class Library
  def self.configure
    yield(configuration)
  end

  def self.configuration
    @configuration ||= Configuration.new
  end
end
```

Now consumers of the library can:

```ruby
Library.configure do |config|
  config.title = "My Custom Title"
  config.text = "My Custom Text"
end
```

Any place we reference `Library.configuration.title` and `Library.configuration.text` will return the configured values.  If none have been configured then our defaults in `Configuration#initialize` will be returned:

```erb
 <h1><%= Library.configuration.title %></h1>
 <p><%= Library.configuration.body %></p>
```

This all works for two reasons:

1. We memozied the configuration with `@configuration ||= Configuration.new` so any configured changes applied to `@configuration` will be returned.

2. Classes in Ruby are just instances of `Class` referenced by a constant, which can't be changed (That's not entirely true, but we'll save that explanation for another time).  This means that no matter where we reference `Library.configuration` we know we're getting the configured configuration.

*Fun Fact: These two lines do the same thing.  Maybe we'll explore this in a future blog post:*

```ruby
class Library; end
Library = Class.new
```

This is the exact pattern I used to implement the [Flipper::UI` configuration](https://github.com/jnunemaker/flipper/commit/3ced5fb32021a428b369cdabb83afc7e2c64c22c) and users seem find it pretty cool.
