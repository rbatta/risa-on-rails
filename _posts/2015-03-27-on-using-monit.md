---
title: "Integrating Monit to Slack"
layout: post
date:   2015-03-30
tags: [slack, monit, devops]
categories: devops monit slack
permalink: /integrate-monit-with-slack/
---

The conversation of what software are we going to choose for monitoring our apps and servers came up at work. We've all used New Relic and Nagios before, but not really Monit. New Relic is free to a certain point, but can get hella expensive if you've got multiple apps and want to check many different things. Both Nagios and Monit are open-source, which equates to _FREE_. Perfect for dev shops and small businesses.

Since I like to read a lot of different articles while researching things, I kept coming across Monit as the go-to tool for monitoring instead of Nagios. Plus my boss had some bad experiences with setting it up before. Well, my Google-fu is fairly strong, so I decided to give Monit a try.

I admit, setup was fairly simple.  Currently our stack is LEPPR (Linux Nginx Passenger Postgres Rails), not that it matters though. So real quick, here were the steps I used to set it up, before going to the meat of the article: integrating with Slack.

{% highlight bash %}
$ sudo apt-get install monit
{% endhighlight %}

The file you want to edit will be `/etc/monit/monitrc` which is the main config file. Comment out the bits about the httpd section, about mid-way thru, and add in your info as needed. As always, geared for Digital Ocean:

    set httpd port 2812 and
      use address IP_ADDRESS       # Put your droplet IP here
      allow 0.0.0.0/0.0.0.0        # allow anyone to connect to the server and
      allow admin:'P@ssw0rd'       # require user and password for login

What this does is allow you to hit up your IP address in the browser and see how things are looking: http://IP_ADDRESS:2812.

Right on.

Now, let's integrate!

### On Slack
If you're running the desktop app, click on the channel name's `v` down-button and choose Configure Integrations.

* On the web page, choose "_Or, make your own!_" which should take you to /services/new#diy.
* Click Add for __Incoming Webhooks__
* You can choose a channel or make a new one. I created one called `#monitoring` because I'm not original.
* Click Add Integration.

So now you've got a webhook URL that looks something like this: https://hooks.slack.com/services/HASH/Hash/LongHash.  You can test out if it works by scrolling to the bottom and picking the integration settings, choosing what you want for description, custom name, etc. Test it out with the `curl` command that they give you.  It's JSON that drives the entire thing. 

### On Monit
Now, back to the box. We're rubyists here so this is how we roll. We're going to create a file called `slack.rb` in your `/etc/monit/` directory.

{% highlight bash %}
$ sudo touch /etc/monit/slack.rb
$ sudo nano /etc/monit/slack.rb
{% endhighlight %}

Now in this file, insert this:

{% highlight ruby %}
#!/usr/bin/ruby

require 'net/https'
require 'json'

uri = URI.parse("https://hooks.slack.com/services/HASH/Hash/LongHash")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true
request = Net::HTTP::Post.new(uri.request_uri, {'Content-Type' => 'application/json'})
request.body = {
    "channel"    => "#monitoring",  # channel name
    "username"   => "Monit Bot",    # you can name the bot whatever
    "icon_emoji" => ":trollface:",      # because why the hell not!
    "text"       => "[#{ENV['MONIT_HOST']}] #{ENV['MONIT_SERVICE']} - #{ENV['MONIT_DESCRIPTION']}"
}.to_json
response = http.request(request)
puts response.body
{% endhighlight %}

The cool thing about this is that you can pick whatever slack icon you want to use OR you can even upload your own for an emoji and use that. Obviously you'll have to change the `uri`, `channel`, `username` and the like to what applies to you.

Do a sanity check at this point to make sure everything's correct. Run the ruby script real quick-like.

{% highlight bash %}
$ ruby /etc/monit/slack.rb
{% endhighlight %}

Did you get a notification in Slack?  Great, now make that into an executable file!

{% highlight bash %}
$ sudo chmod +x /etc/monit/slack.rb
{% endhighlight %}

You will need to invoke this file whenever something happens, so to test it out, we're going to create an nginx file to monitor nginx. This will be kept in `/etc/monit/conf.d/` directory as a file called `nginx`.

{% highlight bash %}
check process nginx with pidfile /var/run/nginx.pid
  start program = "/usr/sbin/service nginx start"
  stop  program = "/usr/sbin/service nginx stop"
  if failed host 127.0.0.1 port 80 then restart
  if changed pid then exec /etc/monit/slack.rb    # this is our sanity check
  if failed host 127.0.0.1 port 80 then exec /etc/monit/slack.rb else if succeeded then exec /etc/monit/slack.rb
  if cpu is greater than 40% for 2 cycles then exec /etc/monit/slack.rb else if succeeded then exec /etc/monit/slack.rb
  if cpu > 60% for 5 cycles then restart
{% endhighlight %}

The basic format is that whenever you need to invoke an alert, instead of a `then alert`, you are changing it to execute the slack.rb file. You __must__ specify the full path to slack.rb. Pretty much any time you see a `then alert`, change it to `then exec /etc/monit/slack.rb`. :)

Check to ensure file is correct, reload monit to accept the changes, and then stop/start nginx.

{% highlight bash %}
$ sudo monit -t       # if errors, fix!
$ sudo monit reload
$ sudo service nginx stop
$ sudo service nginx start
{% endhighlight %}

There's a few second delay but check out slack! You should see the message with your info all in there. HURRAY!

This article used an Ubuntu 14.10 droplet.

Level up +3

***

Questions? Comments? Please hit me up at risaonrails (at) gmail.com!
