---
title: "The Pain of RVM, Foreman, and Upstart"
layout: post
date: 2015-04-08 21:44:53   
tags:
- thing
- thing
categories:
- rails
- devops
permalink: /:title/
summary: "The documentation for all of this didn't really help me..."
---
In yet another ongoing adventure thru DevOps land, I've found myself scouring the internet for all its resources. Either I've got to get better at my Google-fu, the things I'm doing are so super basic no one else has this problem, or no one has yet to encounter this problem. Good times. _/sarcasm_

I ended up doing something similar to Denis, except used Ruby wrapper per RVM docs. This was really really annoying tbh,  but according to `top -c` and `shift-M` it's working.  I hope this helps someone else.

My setup is: Digital Ocean, Ubuntu 14.10, Rails 4.0.x, Ruby 2.0.x, RVM 1.26.10. My Procfile is only for background jobs. The deploying user is 'rails'.  I have a gemset - called "ruby-2.0.0-p598@rockin" - for my app called "rockin" since I run multiple apps on the box.

**Adding in the absolute `PATH` to bundle did NOT work for me.**

Here's what I did. I realize it's a bit roundabout but I had to figure out which worked and which didn't. :

1. Create rvm wrapper per [docs][1].  _user is rails_

        rvm alias create rockin ruby-2.0.0-p598@rockin

2. Create .env file for RAILS_ENV and PATH for bundle (`whereis bundle`)

        RAILS_ENV=production  
        PATH=/usr/local/rvm/rubies/ruby-2.0.0-p598/lib/ruby/gems/2.0.0/bin/bundle

3. Attempt to foreman export to upstart

        rvmsudo foreman export upstart /etc/init -a rockin -u rails

4. Decided to tail the logs because of the bundle issue. _user is root_

        tail -f /var/log/upstart/rockin-worker-1.log

5. Run it

        sudo start rockin-worker

5. Cry because of the stupid error about bundle.  (`/bin/sh: 1: exec: bundle: not found`) :sadface:

5. Change the upstart files manually. The file I needed to edit was `rockin-worker-1.conf`. Since most of the thing was pretty formatted and had what I needed, I only removed the `PATH=` and changed the exec lines to truly point to bundle using the wrapper.

        start on starting rockin-worker
        stop on stopping rockin-worker
        respawn

        env PORT=5000
        env RAILS_ENV='production'

        setuid rails

        chdir /home/rails/rockin

        exec /usr/local/rvm/wrappers/rockin/bundle exec rake environment resque:work QUEUE=*

  [1]: https://rvm.io/deployment/init-d "docs"

{% icon fa-angle-double-up %} Level up +15

Hey, if you found that this worked for you, please upvote my answer on [StackOverflow](http://stackoverflow.com/a/29530087/2464546)!

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com