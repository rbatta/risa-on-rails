---
title: "Sandboxing Rails"
layout: post
date:   2015-04-01
tags: [rails, sandbox, console]
categories: rails console sandbox
permalink: /sandboxing-rails/
---

Hey, did you know that there's an option called `--sandbox` that you can append to console? 

Yuuuup, you can run `rails console --sandbox` when you load up your console. What's it for? Simple -- to ensure that any changes you make to the database are _rolled back_ on exit.

### Why use this?

Well, what if you're going into your production box and need to poke around with the data. The site's live, y'know. It's great to be able to go into a production box, run `rails c` and not have to worry that you may have accidentally deleted an entry or updated a value. Think of it as an extra safety net. Here's what it'd look like.

{% highlight bash %}
$ RAILS_ENV=production bundle exec rails c --sandbox
{% endhighlight %}

Once in, you can do the usual CRUD actions, even run `.save!`, and not have to worry about how the actual production database is affected. Here's proof.

{% highlight ruby %}
❯❯❯ RAILS_ENV=production rails c --sandbox
Loading production environment in sandbox (Rails 4.0.10)
Any modifications you make will be rolled back on exit
irb(main):001:0> User.first
=> nil
irb(main):002:0> u = User.create
=> #<User id: nil, name: nil, email: nil, created_at: nil, updated_at: nil, password_digest: nil, remember_token: nil, admin: false>
irb(main):003:0> u.name='risa'
=> "risa"
irb(main):004:0> u.email='test@example.com'
=> "test@example.com"
irb(main):005:0> u.password='password'
=> "password"
irb(main):006:0> u.password_confirmation='password'
=> "password"
irb(main):007:0> u
=> #<User id: nil, name: "risa", email: "test@example.com", created_at: nil, updated_at: nil, password_digest:    "$2a$10$qmgIZE5IqicXfOc6WxOb4.s7rVm4uLAciYJnAnvukJuu...", remember_token: nil, admin: false>
irb(main):008:0> u.save!
=> true
irb(main):009:0> User.first
=> #<User id: 1, name: "risa", email: "test@example.com", created_at: "2015-03-24 17:53:17", updated_at: "2015-03-24 17:53:17",     password_digest: "$2a$10$qmgIZE5IqicXfOc6WxOb4.s7rVm4uLAciYJnAnvukJuu...", remember_token:    "eb878de3cfc54eda3df1ea974ac6a49a35a52f47", admin: false>
irb(main):011:0> exit

❯❯❯ RAILS_ENV=production rails c --sandbox
Loading production environment in sandbox (Rails 4.0.10)
Any modifications you make will be rolled back on exit
irb(main):002:0> User.first
=> nil
{% endhighlight %}

BOOM. Sweet sweet success. No prod box will scare you from now on.

{% icon fa-angle-double-up %} Level up +1

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com