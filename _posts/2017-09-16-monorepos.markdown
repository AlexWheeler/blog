Like many technology companies we happily use Git to decentralize our development workflow. Browsing through the VTS organization on Github you’ll notice a similar trend. Each of our repositories tracks one logical project.  This works really well for us and countless other companies around the world.  However, just because this is the optimal way to organize our projects, doesn’t mean its the best approach for every project. The alternative to this approach is the monorepo.

Flipper is a Ruby gem we use extensively at VTS to turn features on or off for a set of users. Like many of our projects, Flipper chose to use Git as a key part of its development workflow, but unlike VTS, its repository is organized as a monorepo. The rest this post we’ll walk through a real-world issue opened on the Flipper repo to answer two important questions — What is a monorepo, and when would I use one?

## Two Weeks Earlier...
*Thursday, 09-14-2017, 9am*

With a warm cup of coffee and visions of glory, Orbyt is ready to use Flipper for the very first time. Following a provided example, he gem installs Flipper and smiles to himself as the all too familiar characters stream down the monitor in front of him.

`1 gem installed`

Its time. Orbyt initializes a new Flipper instance using the ActiveRecord adapter:

```ruby
require 'flipper/adapters/active_record'

adapter = Flipper::Adapters::ActiveRecord.new
flipper = Flipper.new(adapter)
```

`uninitialized constant Flipper::Adapters::ActiveRecord`

The dreaded uninitialized constant.  As much as we'd like to believe the `Flipper::Adapters::ActiveRecord` class is defined, the Ruby interpreter insists otherwise - and Ruby is always right.

At this point in the story Orbyt opens a really great issue on the Flipper repo, which inspired me to write this post. How can it be that this adapter class is undefined?

Browsing through the [Flipper](https://github.com/jnunemaker/flipper) repo you see it right there, [Flipper::Adapters::ActiveRecord](https://github.com/jnunemaker/flipper/blob/master/lib/flipper/adapters/active_record.rb).

![active_record]({{ site.baseurl }}/assets/monorepos/active_record_full.png)

We've all been here.  You're installing unfamiliar software that you expect to work, but for whatever reason it isn't.  Is this library broken?  Is my computer broken? What day is it?

## Down the Rabbit Hole

A great first step in this case is to locate and open the installed library to see the actual source that's running.  Ruby developers can do this with `bundle show gemname`, where *gemname* is the name of the gem you'd like to locate.

*note: I'm using RVM for managing my ruby versions, your path might look a little different*

`bundle show flipper`

`=> /Users/Alex/.rvm/gems/ruby-2.3.3/gems/flipper-0.10.2`

Change into the flipper directory

`cd /Users/Alex/.rvm/gems/ruby-2.3.3/gems/flipper-0.10.2`

and list the adapters:

![local_adapters]({{ site.baseurl }}/assets/monorepos/local_adapters.png)

We see 7 ruby files corresponding to 7 adapters.  Let's compare this with the adapters in the Flipper
repo on GitHub:

![remote_adapters]({{ site.baseurl }}/assets/monorepos/remote_adapters.png)

The Flipper we downloaded only has 7 adapters, none of which define `Flipper::Adapters::ActiveRecord`, while the Github repo lists 14.

## Enter Monorepo

 Flipper uses what's known as a monorepo - a single git repository to track multiple projects.  This approach works really well in Flipper's case.  Flipper has a core API that calls into [adapters](https://github.com/jnunemaker/flipper/blob/master/docs/Adapters.md) when it needs to interact with a datastore.  Each adapter implements the same interface so they are completely interchangeable.

* `features` - Get the set of known features.

* `add(feature)` - Add a feature to the set of known features.

* `remove(feature)` - Remove a feature from the set of known features.

* `clear(feature)` - Clear all gate values for a feature.

* `get(feature)` - Get all gate values for a feature.

* `enable(feature,` gate, thing) - Enable a gate for a thing.

* `disable(feature,` gate, thing) - Disable a gate for a thing.

* `get_multi(features)` - Get all gate values for several features at once.

Let's see an example:

Here we're using the ActiveRecord adapter, which knows how to query a PostgreSQL database using ActiveRecord.  We enable a feature for a given user and the adapter takes care of storing this in the database.

```ruby
adapter =  Flipper::Adapters::ActiveRecord
flipper = Flipper.new(adapter)
flipper[:cool_feature].enable(user)
```

Tomorrow the team decides document databases are much cooler and switches to MongoDB.  Instead of having to rewrite all of the flipper code we simply swap out the ActiveRecord adapter for the Mongo adapter and everything continues to function as normal - the only difference being where Flipper puts the data, this time in a MongoDB instance.

```ruby
collection = Mongo::Client.new(["127.0.0.1:27017"], database: 'testing')['flipper']
adapter = Flipper::Adapters::Mongo.new(collection)
flipper = Flipper.new(adapter)

flipper[:cool_feature].enable(user)
```

The `Flipper` class provides a simple public interface, which is used in favor of interacting with adapters directly. This means you'll always develop a Flipper adapter alongside the Flipper core API code,  which brings us to the first benefit of monorepos:

**1. Dependency Management**

Instead of requiring developers to figure out how to include the necessary Flipper code in their own adapter repo, they can develop the adapter in the Flipper repo with full access to the code base.  This reminds me of the second benefit to monorepos:

**2. Easy Navigation**

While building an adapter you'll surely need to refer to various sections of the code. Navigation much easier when everything is in one project.  I'd prefer to `ctrl-p` for something over searching through multiple repos any day.

**3. Simple Setup**

If you wanted to build a Flipper adapter right now (and you should!) all you'd need to do is:

* `git clone https://github.com/jnunemaker/flipper.git`

* `cd flipper`

* `vim spec/flipper/adapters/your_awesome_adapter_spec.rb`

```ruby
# your_awesome_adapter_spec.rb

require 'flipper/spec/shared_adapter_specs'

it_should_behave_like 'a flipper adapter'
```

`rspec spec/flipper/adapters/your_awesome_adapter_spec.rb`

The Flipper repo already has an RSpec shared adapter spec to code against to make sure your adapter works as expected.  Red, green, refactor your way to an open PR.

A few days from now, once your awesome PR is merged, let's imagine [John](http://github.com/jnunemaker) decides to add a new method to the adapter interface.  With this change comes the work of implementing this method for all existing adapters, which brings us to the fourth benefit of monorepos:

**4. Cross-repo Changes**

Being an open source project, Flipper leans on the global community of developers to move it forward.  Multiple people from around the world will be working together to update the many adapters. Want to see when Sarah updated the [Redis adpater](https://github.com/jnunemaker/flipper/tree/master/docs/redis)? No problem.  Just check the git history.  We have a monorepo, remember, so all changes are tracked within the same git repository.  No need to switch between projects to get the whole picture of the project's history.  This brings us to the fifth, and our final, benefit to monorepos:

**5. Common Versioning**

We check git and see that all adapters have been updated.  Now comes the time to release a new version into the wild.  As a Flipper user I'd love to use these updated adatpers, but which adapter versions support this new method?  Mongo adpater 2.1.5? ActiveRecord adapter 8.2.8? Memcached 1.6.2?  Let's keep it simple!  All projects in the monorepo share a common version number, statically defined in a single place for reference!

```ruby
# lib/flipper/version.rb
module Flipper
  VERSION = '0.11.0.beta7'.freeze
end
````

Flipper 10.2 works with flipper-active_record 10.2, flipper-mongo 10.2, flipper-redis-10.2, etc.

By now we should have a good idea why Flipper uses a monorepo, but we haven't answered `Orbyt's` question.

Why is `Flipper::Adatpers::ActiveRecord` `undefined`?

This is where the monorepo really shines.  You see, flipper-active_record depends on [ActiveRecord](https://github.com/rails/rails/tree/master/activerecord), flipper-mongo depends on a [mongo ruby driver](https://github.com/mongodb/mongo-ruby-driver), flipper-redis requires the [ruby redis client](https://github.com/redis/redis-rb)...you get the point.  Just because we develop these together doesn't mean we have to distribute them together.  Flipper users install the core Flipper gem with `bundle install flipper` and then individually download any adapters they'll need.

*e.g.*
`bundle install flipper-active_record`

The key here is that `bundle install flipper` **does not** download any adapter dependencies it doesn't need.  Let's see how this works.

## Anatomy of A Gem

Gems are ruby libraries - packaged code that can be shared with the world.  A gem has three components:

1. code

2. documentation (ideally)

3. gemspec

### gemspec

The gemspec is a file stored in the root directory of a gem that specifies information about the gem, such as its name, version, description, authors, etc.  But this isn't all that a gemspec can specifiy.  A gemspec can configure many options, most importantly in our case, `files`- which files to include in the packaged gem.

*For a full list of options check out RubyGem's [reference](http://guides.rubygems.org/specification-reference/)*.

You'll find Flipper's gemspec, like all gemspecs, in the root directory.  You'll also notice some other gemspecs that have names resembling many of the adapters mentioned earlier.  You might see where we're headed with this...

![gemspecs]({{ site.baseurl }}/assets/monorepos/gemspecs.png)

Opening flipper.gemspec and removing unnecessary code for this conversation we'll focus on how Flipper is able to ignore unnecessary code when packaging the gem.

```ruby
plugin_files = []

Dir['flipper-*.gemspec'].map do |gemspec|
  spec = eval(File.read(gemspec))
  plugin_files << spec.files
end

ignored_files = plugin_files
ignored_files << Dir['script/*']
ignored_files << '.travis.yml'
ignored_files << '.gitignore'
ignored_files << 'Guardfile'
ignored_files.flatten!.uniq!

ignored_test_files = plugin_test_files
ignored_test_files.flatten!.uniq!

Gem::Specification.new do |gem|
  gem.files         = `git ls-files`.split("\n") - ignored_files + ['lib/flipper/version.rb']
end
```

We start with an empty array `plugin_files`.  `Dir['flipper-*.gemspec']` returns an array of all file names that start with `flipper-` followed by any number of characters, `*`, and end with `.gemspec`.  To better demonstrate what's happening we could evaluate that line and replace it with its result.

```ruby
['flipper-active_record.gemspec', 'flipper-api.gemspec', ...].map do |gemspec|
  spec = eval(File.read(gemspec))
  plugin_files << spec.files
end
```

Continuing down the file we come across one of the coolest uses of Ruby's `Kernel#eval` I've come across in the wild. `eval(str)` evaluates the ruby expressions in *str*. Gemspecs might have the *.gemspec* file extension, but don't let that fool you, they're plain old ruby files that can be read into a string and evaluated.

```ruby
spec = eval(File.read(gemspec))
```

By evaling the gemspec we get a reference, `spec`, to the last ruby expression executed in the file `Gem::Specification.new do ... end`.

We push all of the yielded gemspec's files into `plugin_files`.

```ruby
plugin_files << spec.files
```

The rest of the file is pretty straightforward.  Ignore travis (CI) configuration, .gitignore, anything under scripts/, etc.

```ruby
ignored_files = plugin_files
ignored_files << Dir['script/*']
ignored_files << '.travis.yml'
ignored_files << '.gitignore'
ignored_files << 'Guardfile'
ignored_files.flatten!.uniq!

ignored_test_files = plugin_test_files
ignored_test_files.flatten!.uniq!
```

Take all of these files we'd like to ignore and remove them from the the total files tracked in this git repository.  The result is a list of all the files that will be bundled with the published Flipper gem on RubyGems.

```ruby
gem.files = `git ls-files`.split("\n") - ignored_files + ['lib/flipper/version.rb']
```

And there we have it. Flipper::Adapters::ActiveRecord is undefined because although the source exists in the monorepo, the bundled flipper gem does not include it when published. Thanks monorepo.

Big shoutout to `Orbyt` for opening an awesome issue and submitting a [PR](https://github.com/jnunemaker/flipper/commit/5761282a4b47acba7faee4c706be7227d31467d5) that improves documentation for all flipper users.
