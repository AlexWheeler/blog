![datacenter]({{ site.baseurl }}/assets/ssh-csv-fun/datacenter.png)

One of the most interesting parts about working at a company vs. hacking on side projects is that you are truly working on a team.  Yes, you could argue that participating in hackathons with friends counts as working on a team and I agree that it does, however, most companies need much more than just engineers to build a product.  Without a talented sales team, marketing team, etc.  working alongside an engineering team it would be near impossible to grow a company.  This is why ChallengePost feels like a true team to me.  Yes, there is an amazing engineering team, however there is a just as talented number of teams working every day on the business side of the product who have shaped ChallengePost into the company it is today.

As you probably, know most software companies have a ton of data.  If you think about it at the end of the day your product is just a bunch of data spread out over the interwebs.  A lot of this data is very valuable to both your team and your clients.  Because of this, quite often we find the business team requesting data in one form or another.  This could be for a client, an internal meeting, or maybe they just think it is funny to test out our scripting skills (most likely one of the first two, but you never quite know).  So, this takes us to servers.  I’d like to talk about how you can access, find, and retrieve data from your servers that can then be formatted in which ever way you would like (actually you most likely don’t have a choice and it is up to the person requesting it)

First off, what is a server? Well, understanding servers is actually pretty simple.  Servers are just computers that sit on a network waiting for clients to ask for things.  Servers store all of your data and can be accessed quite easily through a cryptographic network protocol (fancy, right?) known as Secure Shell or SSH.  SSH requires three things:

1. public key

2. private key

3. passphrase

First you must generate your keys, which is done by using a Unix utility called
ssh-keygen from your command line.

![ssh-keygen]({{ site.baseurl }}/assets/ssh-csv-fun/ssh-keygen.png)


![public-key]({{ site.baseurl }}/assets/ssh-csv-fun/public-key.png)

Choose a passphrase and a file to save them into.  You should now have a private and public key stored in ~/.ssh/

Given that you have a server living somewhere on the internet You will give your public key to it and hold onto the private key.  Remember that everything on the web has an address known as an IP address so by running the ssh command from the command line with your username and server’s address you can say “Hey, server, it’s me let me in”

`ssh username@ip_address`

The server will ask you for your passphrase:

![passphrase]({{ site.baseurl }}/assets/ssh-csv-fun/passphrase.png)

If you have entered the correct passhphrase the server will generate and send a large encrypted string to your computer that only the corresponding private key knows how to successfully decrypt.  When your private key accomplishes this task you are succesfully given access to the server and can navigate it from the command line just as you would your PC.

![identity-added]({{ site.baseurl }}/assets/ssh-csv-fun/identity-added.png)

So, now for the fun stuff.  Like I said, many times throughout the week the business team asks for data.  Many of these requests are the same exports every week and for those we have saved the scripts to do so, however a lot of the time we must write these scripts ourselves.  I’ll go over one example of something I had to do the other day.  My Product Manager needed a had a CSV file with lots of rows of software projects that users had uploaded.  We needed to go into one of our staging servers, find all the projects and get some other info about them and save a new CSV file with the new output.  Sounds exactly like something a computer would be good at.  From a high level we already know a few things.

1. We know that we have a CSV containing a lots of software projects.

2. Each row contains at least one column that will have a unique value to that object…definitely something we can query with.

3. Ruby has an awesome CSV library

The plan: read in the original CSV file, parse each row for a unique value, query database for objects by this value, write to a new CSV file with the new attributes that the business team needs.

Let’s pretend the original CSV looks like this: each row is a different project object and column B is the unique value we can use to query by (could be a name, id, etc.)

![csv]({{ site.baseurl }}/assets/ssh-csv-fun/csv.png)

First thing is to get this file on our server so that we can read from it.  First thing you’re going to need to do is to cd into the directory that your app is stored in on your server.  Once in this directory you can use a cool tool called [Wget](https://www.gnu.org/software/wget/) - a cool tool for retrieving files using HTTP, HTTPS, and FTP that can be run from your command line.

`wget http://url_to_file/original.csv`

This will download the file into the current directory you’re in on the server.  Now we can read from it using Ruby’s CSV library.  Here’s an example script that I threw together to show how you could accomplish this sort of task.

```ruby
unique_values = []

CSV.foreach("original.csv") do |csv|
  unique_value = row[1]
  unique_values << unique_value
end

CSV.open("new.csv", "wb") do |csv|
  unique_values.each do |unique_value|
    Software.where("unique_value = ?", unique_value).find_each do |software|
      csv << [software.first_attr, software.second_attr, software.third_attr]
    end
  end
end
```

All this does is create a new empty array called unique_values.  The first block reads in a file named original.csv (the one we downloaded using wget) and for each row in the CSV file it adds the second value (Arrays are 0 indexed) to the unique_values array.

In the second block we create and open a new file called new.csv in write mode.  We iterate over all of the unique_values and query our database for software objects with this value.  We place three attributes of each software object into an array and each time the loop runs it will add these to our new csv file (each array in csv is a row).  Open up your rails console and run this file.  We should now have a file named new.csv saved onto our server that looks something like this (given columns A and B we wanted the same two values the original CSV gave us)

![csv2]({{ site.baseurl }}/assets/ssh-csv-fun/csv2.png)

Now we just need to get the file back onto our PC so we can send it back to the business team.  A simple and secure way to do this is with a protocol called Secure copy or SCP.  From the command line in your root directory on your PC enter scp followed by your username and ip address of your server, the path to the file you would like to transfer, and the path to where you would like to save this on your PC.

`scp username@server_ip_address:/path/to/file/new.csv /Users/Alex`

The last thing to do is to remove the files from your server, email the file to the business team and grab a beer because you did great!
