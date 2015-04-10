---
title: "Creating new old-version Rails apps"
layout: post
date: 2015-04-02   
tags: 
- rails
- tip
categories: 
- rails
- development
- tip
permalink: /:title/
summary: "Quick tip on creating a Rails app using an older version of Rails than the one you've got already"
comments: true
---

As always, a work story. Here at Team DevOps, we've been tasked to create a couple of Rails apps using different Ruby versions. After all, why not? We're also Rails developers too, even if we don't look like it. Admittedly, it's been so long since I've started a Rails app from scratch I almost forgot how to do `rails new`... Yeah, only slightly embarrassed.

I'm more familiar with Ruby 2.0.0 and Rails 4.0.x than 2.2.x and 4.2.x, so I decided to make my app using the older versions. Of course, I've since updated my rubies and rails versions, so without thinking much about it, I did `rails new`. Figured I could just bundle update when I switched ruby/rails versions.

Oh god, that was a terrible, TERRIBLE mistake.

![Huge mistake](http://www.nicknotas.com/wp-content/uploads/2013/04/Ive_Made_a_Huge_Mistake.jpg)

What got created was an awesomely standard Rails 4.2.0 app with Ruby 2.2.0. No problem. In my Gemfile I added in `ruby 2.0.0` and bundle updated and thought nothing of it. I moved on and edited my database.yml file and decided to add some of my standard gems I'd be using for TDD: guard and spork. But since it was rails 4.2, I wasn't sure how my guard/spork setup would work, since I was plugging and chugging from an old 4.0.x app I had. Well, whatever.

I'll just change rails 4.2.x to rails 4.0.x! No problemo!! Lemme bundle update and then run `guard init rspec`. 

![Kaboom!](http://stream1.gifsoup.com/view/897102/mushroom-cloud-o.gif)

What. Have. I. Done. Guard was complaining that I had started things up with a newer version so it wouldn't be able to use the version 4.0.13 wanted (2.6). Not only that but `rails g anything` wasn't working either. Geezus. So I completely uninstalled guard and then installed only the correct version, but even then the Guardfile created seemed weird. Rails didn't let me install rspec either without complaining. Not to mention Rake. Oy vey. How much had changed between Rails 4.0 and 4.2??! What is this business about `activejob_railties`??

OK, there were some major changes that I haven't really been paying attention to. ActiveJob was a new addition to 4.2. On top of that, the `config/initializers/application.rb` file format had changed from referencing the `AppName::Application...` to `Rails.application...` Then there were the additional railties that were added in. This is just the tip of the iceberg...

Yeah let's queue up the whole `I don't know what I'm doing` meme here. Seriously. At this point the only thing I could sensibly do was to completely start over and create a Rails 4.0.x app. But how would I do that?? 

The Rails guides didn't have any indication on how to reference versions when creating it, so off to Google and StackOverflow.

### The way of the old rails warrior
I can just refer an older Rails version when doing `rails new`! Since I wanted 4.0.13, I did this: `rails 4.0.13 new appname`. This did not work.

My syntax was incorrect, and maybe I need rails 4.0.13 installed. I also wanted to build the app with ruby 2.0.0 so in the end these were the correct steps. (I use rbenv on my machine, and apparently didn't have gems for p643 so used p451.)

{% highlight bash %}
$ rbenv global 2.0.0-p451
$ gem install rails 4.0.13
$ rails _4.0.13_ new appname
{% endhighlight %}

Please note the `_` surrounding the version. This is key! Once I did that and installed the app, everything was set properly. I created a rails 4.0.x app that was locked in with ruby 2.0.0. FINALLY. Bundle installed, set up guard and spork and rspec and I was on my merry way to coding heaven. I commited that sucker faster than a speeding bullet.

Now, how do I write new models again...?

{% icon fa-angle-double-up %} Level up +1 for super obscure learnings.

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com