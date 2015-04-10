---
layout: post
title:  "A Rails Double App Setup on Digital Ocean"
date:   2015-03-29
tags: 
- rails
- nginx
- digital ocean
- double setup
- unicorn
categories: 
- rails 
- digital ocean 
- nginx
- guide
permalink: /a-rails-double-app-setup/
summary: "A how-to guide to set up 2 rails apps using the Digital Ocean 1-click Rails template"
comments: true
---

I recently started using Digital Ocean because of work. Dat doing the devops bidness. Right. If you're unfamiliar with Digital Ocean, they are a virtual server provider that uses SSD hard drives and a really cheap pricing plan.  Since it's bare bones, you're responsible for everything, from security, to installation of whatever. That being said, they DO (haha) provide some templates to make things easier. One of them is their Rails 1-click template, which has Nginx, Unicorn, and MySQL already installed. Right on! 

Hey, if LAMP stack is Linux Apache MySQL PHP and LEMP is the same but Nginx, then is this setup called LEMUR? Linux Nginx MySQL Unicorn Rails. And if it's Postgres, then it'd be LEPUR? Well, I'm calling them this from here on out!

That being said, what happens when you actually want to put TWO apps on a DO rails-template box?  There's a lot of weird sometimes-reverse engineering going on with that, but overall, it's totally possible to do. There isn't any documentation on this, so if you run into this scenario, hopefully this helps you. 

The method I'm presenting here is the way I had set this up and got working.

___FULL DISCLAIMER___ It's a bit ugly, because I'm still new to this, and at the time no one else had done this before. Since I started writing this post, I realized that our setup was just not sustainable and have moved to using straight-up Ubuntu 14.10 boxes. Also, Ubuntu 14.04 does not support Postgres 9.4. Ew. But that's a topic for another day. 

__TL;DR__ I basically doubled all the unicorn conf files and doubled the server blocks in nginx.

## Digital Ocean 1-Click Rails Template

Start with your standard Ubuntu setup and click the Rails template. Deploy. The DO template comes with a user already created called Rails. They have a great doc on [how to set your own Rails app with their template](https://www.digitalocean.com/community/tutorials/how-to-use-the-1-click-ruby-on-rails-on-ubuntu-14-04-image), but here's my modifications.  Since our apps are on the LEPUR stack, there's a bit of extra work.

###Bash setup
First, start by installing git and Postgres. Skip the Postgres install if you're sticking with LEMUR. I dropped the `sudo` part since I'm logged in as `root` user.

{% highlight bash %}
$ apt-get update
$ apt-get install git
$ apt-get postgresql postgresql-contrib
{% endhighlight %}

Set up git with your name and username. Switch to Postgres user account and create your psql user named `rails` to keep it consistent with the already-created rails user. Gosh that's confusing.

{% highlight bash %}
$ git config --global user.name "YOURNAME"
$ git config --global user.email "YOUREMAIL"
$ su - postgres
$ createuser --interactive -P rails
{% endhighlight %}

Follow the on screen commands for giving your rails postgres-user a password (keep it secret, keep it safe). Don't give it superuser status. Safety first, after all. Exit out of the postgres user account. Set up the rails bash environment (autoload bash shell upon user switch) and create a directory for sockets in /var.

{% highlight bash %}
$ chsh -s /bin/bash rails
$ mkdir /var/sockets
{% endhighlight %}

###Two Rails apps setup
Now we add our 2 apps. For simplicity sake, I'm adding both of them into the /home/rails/ directory, once I empty it out. If you accidentally deleted the entire rails folder, just `mkdir rails` in `/home`.

{% highlight bash %}
$ rm -r /home/rails/*
$ cd /home/rails
$ git clone REPO1_ADDRESS
$ cd /home/rails/REPO1
$ rvm install RUBY1-VERSION-FOR-REPO1
$ rvm use RUBY1
$ gem install bundler
$ bundle install --without development test
{% endhighlight %}

Repeat this for the 2nd app (skipping the 1st `rm -r` step, of course). I assume you have two completely different Ruby versions, as was the case with me: 2.0 and 2.2. H'okayso. You may have noticed me running bundle as `root`. Digital Ocean is a bit weird. They want you to use `root` to bundle install. I know, I know. Whatever. We work with what we've got. Hopefully don't run into any hitches. Instead of using the `--deployment` option, I'm using `--without development test` because I could. (I've had issues with the deployment option.)

Ok this part was the easy part. Like setting up 2 dev environments on your own local machine. :)  Once bundle has been run, chown it to the rails user. The command below should get both repos at once.

{% highlight bash %}
$ chown rails:www-data -R /home/rails
{% endhighlight %}

###Unicorn
I struggled and searched the internet far and wide for someone who took the DO 1-click template to create 2 apps. Their template has a non-standard unicorn and nginx configuration (in terms of files and locations). Again, we work with what we got. The easiest way I found was to duplicate the unicorn files, which are located in two different places: `/etc/default/unicorn` and `/home/unicorn/unicorn.conf`. We'll start with the latter. Out of sheer simplicity, the dupes had 2 appended to the end of it. I also use nano as my editor, so don't judge. :)

_Optional step: If you're paranoid like me, you can git init `/home/unicorn/` directory so you can keep track of changes._

{% highlight bash %}
$ cp /home/unicorn/unicorn.conf /home/unicorn/unicorn2.conf

$ nano /home/unicorn/unicorn.conf

# Edit repo 1's unicorn.conf to look like this:
listen "/var/sockets/unicorn.REPO1.sock", :backlog => 256
worker_processes 4
user "rails"
working_directory "/home/rails/REPO1"
pid "/home/unicorn/pids/REPO1.pid"
stderr_path "/home/unicorn/log/REPO1.log"
stdout_path "/home/unicorn/log/REPO1.log"

# Edit repo 2's unicorn2.conf to look like this:
listen "/var/sockets/unicorn.REPO2.sock", :backlog => 256
worker_processes 4
user "rails"
working_directory "/home/rails/REPO2"
pid "/home/unicorn/pids/REPO2.pid"
stderr_path "/home/unicorn/log/REPO2.log"
stdout_path "/home/unicorn/log/REPO2.log"
{% endhighlight %}

I changed the listen statements to point to sockets as that is [faster than going thru ports](http://www.gotealeaf.com/blog/setting-up-your-production-server-with-nginx-and-unicorn) apparently. Default backlog is 1024, so you can leave or keep it.

Onto the other unicorn config file. This file is the place that commands when the unicorn daemons start and such. Again, start with a copy/rename of the original for the 2nd repo. `Git init` if you're worried about messing things up royally. It really helps.

{% highlight bash %}
$ cp /etc/default/unicorn /etc/default/unicorn2

$ nano /etc/default/unicorn

# Edit these particular sections for Repo 1:

# Path to your web application, sh'ld be also set in server's config.rb,
# option "working_directory". Rack's config.ru is located here.
APP_ROOT=/home/rails/REPO1

# Server's config.rb, it's not a rack's config.ru
CONFIG_RB=/home/unicorn/REPO1.conf

# Where to store PID, sh'ld be also set in server's config.rb, option "pid".
PID=/home/unicorn/pids/REPO1.pid
UNICORN_OPTS="-D --config-file $CONFIG_RB -E production"

# change the RUBY_VERSION_REPO1 based on where
# your rubies are located.
PATH=/usr/local/rvm/rubies/RUBY_VERSION_REPO1/bin/:/home/unicorn/.rvm/bin:/usr/local/sbin:/usr/bin:/bin:/sbin:$
export GEM_HOME=/usr/local/rvm/gems/RUBY_VERSION_REPO1
export GEM_PATH=/usr/local/rvm/gems/RUBY_VERSION_REPO1:/usr/local/rvm/gems/RUBY_VERSION_REPO1
DAEMON=/usr/local/rvm/gems/RUBY_VERSION_REPO1/bin/unicorn

# Env vars needed for the system to run REPO1 because unicorn is silly
# and doesn't understand when Unix has your env vars.
export SECRET_TOKEN=stick_your_secret_token_or_secret_key_base_token_hash
export WHATEVER='env-vars-you-need'
{% endhighlight %}

If you're not sure which ruby version you're using, make sure you do `which ruby` from _within_ the repo.

Repeat these for the `unicorn2` file, changing the values for the 2nd repo.

One very. important. step. left with your unicorn setup. You've got to make a double of your unicorn initializer. That's located in `/etc/init.d/` as `unicorn`.  Copy it and then edit the parts that references Unicorn. 

{% highlight bash %}
$ cp /etc/init.d/unicorn /etc/init.d/unicorn2

# here are the parts I changed in unicorn2
NAME=unicorn2
DESC="Unicorn2 web server"

if [ -f /etc/default/unicorn2 ]; then
  . /etc/default/unicorn2
fi

PID=${PID-/run/unicorn2.pid}

check_config() {
  if [ $CONFIGURED != "yes" ]; then
    exit_with_message "Unicorn2 is not configured (see /etc/default/unicorn2)."
  fi
}
{% endhighlight %}

This allows us to run commands like `service unicorn2 start|stop|restart` much like how the regular unicorn is done. 

Time to restart all the unicorn services.
{% highlight bash %}
$ service unicorn restart
$ service unicorn2 restart
{% endhighlight %}

You can confirm that both are running by checking `top -c` then press `shift-M`. Since we named the 2nd unicorn2, your top should have unicorn and unicorn2 running. Score!

###Nginx
Now for the final bit. Nginx's configuration.

If you opt to `git init` your nginx configuration, do it in `/etc/nginx/` directory rather than `/etc/nginx/sites_enabled/` because any extra files in this directory will be read in by nginx.

I kept both apps' information in the default file as different server blocks, but you can keep them in separate files if you'd like. Whatever floats your boat.

{% highlight bash %}
# anything that isn't https
# becomes https, permanently
server {
  listen 80;
    server_name DOMAIN1.com;
  rewrite ^ https://$server_name$request_uri? permanent;
}

# anything that's not https gets redirected to
# domain2.com
server {
  listen 80;
  server_name DOMAIN2.com;
  return 301 https://DOMAIN2.com$request_uri;
}

# upstream for unicorn on repo 1. make sure
# sockets location is same as in /home/unicorn/unicorn.conf
upstream unicorn-REPO1 {
  server unix:/var/sockets/unicorn.REPO1.sock;
}

# SSL info for repo 1.
server {
  listen 443;
  server_name REPO1.com;

  ssl on;
  ssl_certificate /etc/ssl/REPO1.com.crt;
  ssl_certificate_key /etc/ssl/REPO1.com.key;

  # location of repo 1's public dir
  root /home/rails/REPO1/public;

  client_max_body_size 1024M;

  location / {
    try_files $uri/index.html $uri.html $uri @app;
  }

  location ~* ^.+\.(jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|mp3|flv|mpeg|avi)$ {
    try_files $uri @app;
  }

  # change proxy_pass to the upstream name from above
  location @app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_read_timeout 600;
    proxy_pass http://unicorn-REPO1;
  }

  # for things to be downloaded properly
  location /files/ {
    alias /home/rails/REPO1/some-directory/;
    internal;
  }
}

# upstream for unicorn on repo 2. make sure
# sockets location is same as in /home/unicorn/unicorn2.conf
upstream unicorn-REPO2 {
  server unix:/var/sockets/unicorn.REPO2.sock;
}

# SSL config for server 2
server {
  listen 443 ssl;
  server_name DOMAIN2.com;

  ssl on;
  ssl_certificate /etc/ssl/REPO2.com.crt;
  ssl_certificate_key /etc/ssl/REPO2.com.key;

  # location of repo 1's public dir
  root /home/rails/REPO2/public;
  index index.html index.htm;

  location / {
    try_files $uri/index.html $uri.html $uri @app;
  }

  location ~* ^.+\.(jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|mp3|flv|mpeg|avi)$ {
    try_files $uri @app;
  }

  # change the proxy_pass to point to upstream's name
  location @app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://unicorn-REPO2;
  }
}
{% endhighlight %}

Server blocks are where you put all the good stuff that nginx needs to know with what to do. Since the app we had needed to be automatically changed to https, the listen 80 directive wasn't that important - just needed to redirect to things properly. 

SSL certs and keys were placed in `/etc/ssl/`. If you've got intermediate keys, you will want to have them combined into one giant .crt file.

Check to ensure that your nginx config is syntactically correct. If all good, restart nginx!

{% highlight bash %}
$ nginx -t
$ service nginx restart
{% endhighlight %}

Correct any mistakes. Make sure each directive ends with a `;` !!!

And voila! You're good to go. Switch to the rails user to run your standard rake commands like `rake db:setup` and `assets:precompile`.

### Final thoughts
I am sure you may have some errors for each ruby. Definitely check your logs for nginx and unicorn. When we did this, we had to re-connect the executable-hook for each of the apps since we used different ruby versions for the apps. Stack Overflow had some brilliant articles that helped us troubleshoot. I can't recall which ones they were but if I encounter them again, I'll post it here.

Also, I'm a huge fan of making my life easier, so I created aliases for root to restart services. 

{% highlight bash %}
$ touch .bash_aliases
$ nano .bash_aliases

# copy paste this into the file
alias restart-all='service nginx restart; service unicorn restart; service unicorn2 restart'
alias restart-unicorns='service unicorn restart; service unicorn2 restart'
alias restart-nginx='service nginx restart'
echo '===================================='
echo '       Restart shortcuts            '
echo '===================================='
echo 'restart-all      = restart nginx and both unicorn & unicorn2'
echo 'restart-unicorns = restart both unicorns'
echo 'restart-nginx    = restart nginx'

# source your bashrc file to reload shell
# the echo output should be displayed at the top
$ source ~/.bashrc
{% endhighlight %}

Now you'll be able to run `restart-all` to restart all your services. Woohoo! This is super handy when troubleshooting. And remember, when in doubt, `restart-all` sometimes does the trick. :)

A final word. If you find that your app needs to use Sidekiq or Resque, and you want to use Foreman + Upstart to drive your background workers from your Procfile, I suggest that you do _NOT_ want to use this 1-click template. 

{% icon fa-angle-double-up %} Level up +50

***

Questions? Comments? I know I can make this cleaner, but there was NO article out there that talked about setting up 2 rails apps using the 1-click Digital Ocean rails template. Seriously. But if you're stuck and are following these directions, shoot me an email and let me know! I'll do my best to help if I can.  Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com