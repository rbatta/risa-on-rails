---
title: "Setting up a Rails app on Digital Ocean"
layout: post
date: 2015-04-08 22:10:13   
tags:
- setup
- guide
- rails
- nginx
- rvm
- passenger
categories:
- rails
- devops
- guide
- nginx
- rvm
permalink: /:title/
summary: "A huge huge guide on how I set up a Rails app from scratch"
---
In an effort to keep my sanity (this seems to be a theme amongst my posts; possibly old age??) I am writing my definitive guide to setting up a Digital Ocean box to serve your Rails app. This doesn't involve Capistrano nor chef. The web is inundated with various guides on how to set that up. Use them if you want to do Capistrano deployments. The requirements I have are as follows:

Requirements: 

* Postgres 9.4
* Multiple Rails apps
* Multiple Ruby versions
* RVM
* Workers (for background jobs)
* Ubuntu 14.x

Nice to haves: 

* Passenger 5.0 (codename: Raptor!)

Outside of this, I'm not too worried.

Let's start with the server...

###Ubuntu (and initial Droplet setup)
Man, Ubuntu is fantastic, isn't it? I don't even consider CentOS or RedHat or Fedora or any other *nix flavoured machines when I think virtual server. I automatically think Ubuntu. It's that great. (It's no Mac, but you know...)

If you were not aware, Ubuntu versions have code names associated with them. For instance, Ubuntu 12.04 is called Precise Pangolin (aka Precise) and Ubuntu 14.04 is called Trusty Tahr (Trusty). The `LTS` at the end signifies 'long term support', which means exactly what you think it means. It'll be supported for a very very long time. It's generally the super-duper stable version of that version.

My bone-to-pick. One of my requirements is Postgres 9.4. For whatever reason, Ubuntu 14.04 LTS can __NOT__ use Postgres 9.4!! (Or maybe it's the Postgres guys didn't have a version for 14.04) You're stuck at 9.3.x. Oh yeah, tell me how annoying that is! The only choice then is to use 14.10, Utopic Unicorn (Utopic).  I will cry if 9.5 can't run on Utopic. It's a unicorn after all; it should do anything!

Back to business. On Digital Ocean, set up your droplet with `Ubuntu 14.10 64-bit` at whatever size RAM, tho a 1GB box is good to go. If you didn't add in your SSH Key, you'll get a password for Root. Otherwise, add in your SSH Key to log in via that.

###Initial setup
Log into your droplet (`ssh root@123.123.123.123`), assuming that 123.123.123.123 is your Droplet's IP address. 

####Create a User
Let's set up our main user that will be in charge of handling all the deployments. I'm not creative enough so I'll call mine `risa`. We'll loosely follow the [DO guide they made][1]. 

{% highlight bash %}
$ adduser risa
$ gpasswd -a risa sudo
# put in a password (x2)
# optional to put in name and other info in
# just keep hitting enter to leave blank
{% endhighlight %}

####Create Public Key Authentication (for user)
Again, if you added your SSH Key automatically to your droplet, then you know why this is handy. Never having to add in your password again when logging in is beautiful! These steps create it for the user you just created (e.g. risa). Again, following the above guide's directions.

{% highlight bash %}
$ su - risa
# this switches the user from root to risa
$ mkdir .ssh
$ chmod 700 .ssh
$ nano .ssh/authorized_keys
{% endhighlight %}

At this point, copy paste your `id_rsa.pub` key from your own local computer in `~/.ssh/` into authorized_keys. It'll start with `ssh-rsa AAAAB3N...`.

{% highlight bash %}
$ chmod 600 .ssh/authorized_keys
$ exit  # this should exit you to root
$ exit  # this should exit you out of the box
$ ssh risa@123.123.123.123  # type yes to enter
{% endhighlight %}

Now time to install ALL THE THINGS!

#### >> Installations
Last command given was to log in with your new user name. For me, I logged in as risa. All the commands I run from my account _without `sudo`_ will be as `risa`. If I run commands _with `sudo`_, they will be run as root. You'll see what I mean soon enough. Without further ado.

{% highlight bash %}
$ sudo apt-get update  # put in your password
$ sudo apt-get install git
$ sudo apt-get install postgresql postgresql-contrib libpq-dev
$ sudo apt-get install nodejs
$ sudo apt-get install imagemagick

# and now for redis who isn't like the others
# check redis.io for newest version
$ wget http://download.redis.io/releases/redis-3.0.0.tar.gz
$ tar xzf redis-3.0.0.tar.gz
$ cd redis-3.0.0
$ make
$ make test
$ sudo make install
$ cd utils
$ sudo ./install_server.sh

# choose the default options if you'd like
# default redis port is 6379
{% endhighlight %}

That's good for now. Let's install RVM.

#### >> RVM
RVM is Ruby Version Manager. It allows us to have different versions of Ruby for different apps and keeps our gems separate. For instance, if you have a Ruby 1.9.3 app and a 2.1.5 app, they are bound to have different gems. Those gems may conflict with each other unless they are corralled properly. RVM does that for us really simply. We are going to install the "multi-user" version.

{% highlight bash %}
$ \curl -sSL https://get.rvm.io | sudo bash -s stable
# you're prolly gonna get this complaint about a cert
# so just copy paste that command it wants you to use
# then run the command again
$ \curl -sSL https://get.rvm.io | sudo bash -s stable

# add user risa and root to rvm group
$ sudo adduser risa rvm
$ sudo adduser root rvm

# add the following to your .bashrc file
$ sudo nano ~/.bashrc
# Paste the following in
  [[ -s "usr/local/rvm/scripts/rvm" ]] && . "usr/local/rvm/scripts/rvm" 
$ source ~/.bashrc

# test if rvm is a function
$ type rvm | head -n1    # => rvm is a function

# now install the major ruby versions you want
# e.g. ruby 2.1.5 and ruby 2.2.1
# which are the current newest of 2.1.x and 2.2.x
$ rvm install 2.1.5
$ rvm install 2.2.1
# set one of them as default
$ rvm use 2.2.1 --default

{% endhighlight %}

If you want to install older versions that aren't in the semantic versioning system, when you `rvm install 2.0.0`, it will automatically grab the latest patch version: 2.0.0-p643. You can Google to find the latest released versions for 2.1.x and 2.2.x.

Of course if you intend only to install 1 Rails app on the box, then by all means install that version of Ruby only. Again, I wanted to have multiple apps on the server so multiple rubies were needed.

#### >> Git
Set up yo' git, homey!

{% highlight bash %}
$ git config --global user.name "YOURNAME"
$ git config --global user.email "YOUREMAIL"
{% endhighlight %}

#### >> Postgres
Time to setup Postgres. We're going to create 2 users. One is the default Postgres user, the other is for the user risa, aptly named risa. The overview will be to hop in to root then to postgres, create a password for postgres user, then create the not-superuser risa with a password.

{% highlight bash %}
$ sudo -i         # switches to root
$ su - postgres   # switches to postgres user
$ psql            # hop into postgres
postgres=# \password
# type in your desired password
postgres=# \q
# \q lets you exit out

# create a user with a password
$ createuser --interactive -P risa
  # Enter password for new role:
  # Enter it again: 
  # Shall the new role be a superuser? (y/n) n
  # Shall the new role be allowed to create databases? (y/n) y
  # Shall the new role be allowed to create more new roles? (y/n) n
{% endhighlight %}


I had a lot of help with this. The Digital Ocean tutorials are amazing. Mad props to this guy: [Setting Up Rails on DO](http://winstonyw.com/2014/10/24/setting_up_ruby_on_rails_on_digital_ocean/)

{% icon fa-angle-double-up %} Level up +20

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[1]: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04
[email]: mailto:risaonrails@gmail.com