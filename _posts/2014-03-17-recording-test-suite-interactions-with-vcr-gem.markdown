The other night I got the chance to hang out with a few of the developers at Terrible Labs, a local Rails shop here in Boston.  Over the past year or so I’ve gotten to know some of the team pretty well after meeting [Jeremy Weiskotten](https://twitter.com/doctorzaius) at our local Boston Ruby project night, where he sat down and helped me out with some very… VERY simple Ruby questions I had.  I am forever thankful for his patience, encouragement and hilarious tweets.

Since that night the team has been extremely supportive of me stopping by to hangout, work on projects, and get any questions answered I may have.  I really don’t think I can express how thankful I am for their generosity, patience, and support - truly some of the coolest and most genuine people I know.  When I ran into my friend and [Terrible Labs](http://www.terriblelabs.com/) developer [Alex Jarvis](https://twitter.com/AlxJrvs) at last month’s project night he mentioned he wanted to get a group of local ruby developers together to come in for beers, pizza, hacking, and the chance to get some questions answered by himself as well as [Thomas Mayfield](https://twitter.com/thegreatape) and [Jeffrey Chupp](https://twitter.com/semanticart).  I was beyond stoked to say the least.

I’ve recently been working hard to improve my testing game, so after everyone got the chance to chat a bit about what projects they were hacking on I decided it would be a great time to build out my  text message geolocation project’s test suite.  About five minutes later, when running some integration tests, I already had my first question - why the hell do I keep getting a text message every time my test suite runs? Jarvis’s immediate laugh and enormous grin clearly showed that he recognized this as an awesome learning opportunity.  Without hesitation Thomas immediately connected his laptop to the huge monitor on the wall and began walking us through one of his side projects to demonstrate what was going on and to provide a simple solution.  It turns out that if you are testing code that calls any sort of external service (in my case the Twilio API) an HTTP request is indeed going to get sent. If this request tells the service to fire off a text message, well, you can be sure a text message is going to be sent.  This is not ideal.

There are actually many more problems than merely receiving annoying, repetitive text messages when calling remote services from your test suite.  Just to name a few:

1. Tests failing due to poor connectivity

2. Much slower test suite

3. API rate limits

4. Wasting money (e.g. each Twilio message costs $$$)

Of course, one of the many great aspects of being rails developers is that we have access to an enormous amount of open source gems to help us solve these issues.  One of these gems is VCR.  VCR records your test suite’s HTTP interactions so that you can replay them during future test runs. After configuring VCR you run a test, at which point a file known as a cassette will be created.  Cassettes contain JSON responses from the remote service, which will then be read by your test suite the next time you run the test instead of actually calling the service.  Great, let’s get started.

**Setup Process**

Getting up and running with VCR is pretty straight forward, however I would like to dedicate the rest of this post to walking through the exact steps towards getting everything working with a Rails 4 application and RSpec.

The first step is to add VCR and Webmock to the project’s gemfile - you can disregard the other gems as this just happens to be my gemfile setup at the moment for this project.

`gem "vcr"` - does most of the heavy lifting

`gem "webmock"` - stubs the HTTP request and returns control to VCR, which directs test to read from cassette instead.

1.) Add VCR and Webmock to the project’s gemfile

*Gemfile*

```ruby
group :test, :development
  gem "vcr"
  gem "webmock"
end
```

2.) run bundle to install

`bundle`

3.) configure VCR in spec_helper.rb

This is an overview of my spec_helper.rb file, however we really only care about two parts…

*spec_helper.rb*
```ruby
require "rspec/rails"
require "rspec/autorun"
require "webmock/rspec"
require "vcr"

VCR.configure do |c|
  c.cassette_library_dir = "spec/fixtures/vcr_cassettes"
  c.allow_http_connections_when_no_cassette = false
  c.filter_sensitive_data("<TWILIO_ACCOUNT_SID>") { ENV["TWILIO_ACCOUNT_SID"] }
  c.filter_sensitive_data("<TWILIO_AUTH_TOKEN>") { ENV["TWILIO_AUTH_TOKEN"] }
  c.hook_into :webmock
end
```

The first part is to make sure you have required Webmock and VCR

*spec_helper.rb*

```ruby
require "rspec/rails"
require "rspec/autorun"
require "webmock/rspec"
require "vcr"
```

The second part is to set up the actual configuration.

*spec_helper.rb*

```ruby
VCR.configure do |c|
  c.cassette_library_dir = "spec/fixtures/vcr_cassettes"
  c.allow_http_connections_when_no_cassette = false
  c.filter_sensitive_data("<TWILIO_ACCOUNT_SID>") { ENV["TWILIO_ACCOUNT_SID"] }
  c.filter_sensitive_data("<TWILIO_AUTH_TOKEN>") { ENV["TWILIO_AUTH_TOKEN"] }
  c.hook_into :webmock
end
```

To break this down a little:

`cassette_library_dir =` creates directory to save cassettes to

`filter_sensitive_data =` makes sure cassette uses/displays environment variables such as API keys.  Since a cassettee is a JSON file containing the response from the remote service you don’t want it containing your actual sensitive data - especially if you use source control and have your code anywhere online such as GitHub.

4.) Ready to create first cassette and run tests

your */spec* directory should look similar to this.  */vcr_cassettes* willin itially be empty until we run our first test using VCR.

![dir 1]({{ site.baseurl }}/assets/recording-test-suite-interactions-with-vcr-gem/cassettes-dir-1.png)

5.) Run test

*twilio_request_message_spec.rb*

```ruby
require "spec_helper"

feature "twilio_request_message" do
  scenario "user sends twilio request message to friend" do
    VCR.use_cassette("twilio request message") do
      fill_in "twilio_message_message_from", with: "111-111-1111"
      fill_in "twilio_request_message_to", with: "111-111-1111"
      click_button "Geolocate them!"
      expect(current_path).to eq(twilio_request_message_success_path)
    end
  end
end
```

Whenever you need to use VCR simply wrap your test in this block.  The `use_cassette` method simply takes one parameter - whatever you would like to name your cassette.  The first time you run your test it will send a request to the external service and record this response in a file named whatever you passed in as the parameter to `use_cassette` and save it under */spec/vcr_cassettes*.  Run the test and you should now see a cassette file has been created.

![dir 2]({{ site.baseurl }}/assets/recording-test-suite-interactions-with-vcr-gem/cassettes-dir-2.png)

The file will look something like this:

*twilio_request_message.yml*

![twilio request yaml]({{ site.baseurl }}/assets/recording-test-suite-interactions-with-vcr-gem/twilio-request-yaml.png)

Now anytime you run this test instead of calling the remote service your test suite will simply use this recorded response.  That’s really all there is to it.  Happy Hacking!

Huge thanks again to  Alex,  Thomas and Jeffrey for taking time out of their busy week to hang out and share some knowledge (and beer and pizza).  And also thank you so much Jeremy Weiskotten and the rest of the team at Terrible Labs for being so rad!
