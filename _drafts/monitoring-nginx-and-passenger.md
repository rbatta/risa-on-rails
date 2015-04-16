---
title: "Monitoring Nginx's Passenger"
layout: post
date: 2015-04-15 21:21:01   
tags:
- monit
- passenger
categories:
- guide
- devops
- monit
- passenger
permalink: /:title/
summary: How to use Monit to monitor Nginx's Passenger
comments: true
---
In the ever learning world I'm in, someone had commented on [my post](/integrate-monit-with-slack/) about monitoring Passenger.

That got me thinking and diving deep into the depths of Google to find a solution that was acceptable for those of using Nging to create the Passenger instances.

The most common search result for this was to use the gem [passenger-monit](https://github.com/romanbsd/passenger_monit). However it hasn't been updated in at least 2 years. I am not that ambitious enough to revive this gem for my own purposes. The biggest drawback for using this gem is that the rack PIDs are hardcoded and if the first one dies, Monit will complain. It isn't ideal for my purposes since I run multiple apps on one server and I'd like to keep track of them individually.

HOWEVER~! I have found a way that seems to work.

### The Solution
There's a method called `matching` that monit allows. Essentially monit does a `ps aux | grep 'whatever thing'` to check for a process. [Holy balls, I love you person on ServerFault who answered this](http://serverfault.com/questions/523226/monit-daemonize-non-daemon-process) and I wish I had the reputation to upvote the crap out of that answer. But instead, mad props to you. 

Anyway, after reading that, everything became clear.

Here's what I ended up doing. 

#### The example setup
Assume the following:

* Nginx + passenger (Passenger is spawned via Nginx)
  * there is no passenger PID file that exists
* Rails app named 'Surprise' 
  * nginx configured to point to `/home/rails/Surprise/public`
* Deployer user named 'rails'
* Monit version 5.6 or 5.8 is running
* Digital Ocean droplet for server

Now, if you have not hit Surprise at all, you won't have any passenger instances created. That makes sense. Why spawn a process when it isn't being used? 

The process is brutally simple, hit the app to spawn a Passenger process, then check `top` to see what the name is and create the monit file.

```bash
$ top -c
# hit shift M to sort by memory
# you should see near the top right
# Passenger RubyApp: /home/rails/surprise/public 
# copy paste that

$ sudo nano /etc/monit/conf.d/surprise-pass
```
Now in this file, write in the following:

```bash
check process surprise-pass matching 'Passenger RubyApp: \/home\/rails\/surprise\/public'
  if totalmem > 35% then exec /etc/monit/slack.rb
  if totalmem > 60% then exec /etc/monit/touch_surprise_sh  # restart app
  if changed pid then exec /etc/monit/slack.rb              # sanity check
  if totalcpu > 25% then exec /etc/monit/slack.rb
```
Save and exit. Notice the `\` marks to escape the `/` marks. Because monit does essentially a `ps aux | grep ____`, copy pasting what you find in `top -c` is a great way to get the process match.

A couple of things to note. 1) There is also a new file being executed here. More on this in a bit. 2) We are talking totalmem and totalcpu. Passenger can spin up child processes if it needs it, but I want to look at the total memory consumption of that particular app. Total cpu is not per core but for the whole thing. I also based these off of monit's web interface information. It's very handy to pick out the things you want to monitor in particular.

The new file `touch_surprise_sh` is a super simple bash script that restarts Passenger (and not Nginx) via the `touch` command.

```bash
$ sudo nano /etc/monit/touch_surprise_sh
```
In the file type the following lines:

    #!/bin/bash

    touch /home/rails/surprise/tmp/restart.txt

Alternatively you can make it more verbose and sudo in as your user (rails) to restart Passenger.

    sudo -u rails -H sh -c "touch /home/rails/surprise/tmp/restart.txt”


Save and exit. Make it into an executable file.

    sudo chmod +x /etc/monit/touch_surprise_sh

Do your standard monit sanity check, then reload monit.
```bash
$ sudo monit -t       # fix any problems, obv
$ sudo monit reload
```

Confirm on your web interface that surprise-pass is loaded up and displaying all the goodies you like to see. When you restart nginx, Passenger should lose a PID. It'll alert you on Slack when you hit the app and create a new PID. Right on. :fist:

Congrats! Create a monit conf file for each of your apps and behold your power!

Special thanks to Grant Trevor who got me to delve into the depths of monit and Google :grinning:

{% icon fa-angle-double-up %} Level up +5

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com