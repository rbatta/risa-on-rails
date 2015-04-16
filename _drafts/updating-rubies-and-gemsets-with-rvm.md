---
title: "Updating rubies and gemsets with rvm"
layout: post
date: 2015-04-13 19:39:12   
tags:
- rails
- ruby
- rvm
categories:
- rails
- ruby
- rvm
permalink: /:title/
summary: Oh no! New Rubies came out, and I need to upgrade immediately because security!
comments: true
---
Today, a new security patch addressing an [OpenSSL vulnerability][1] in Ruby was released. This affects Ruby 2.0.0, 2.1, and 2.2. As it had just been released, being the ever vigilant n00b devops engineer, I decided to start upgrading our boxes.  I did not realize that RVM didn't have a binary version of these patches/versions.

```bash
$ rvm install 2.0.0-p645
Searching for binary rubies, this might take some time.
No binary rubies available for: ubuntu/14.10/x86_64/ruby-2.0.0-p645.
```

Thinking nothing of it, I moved on. A couple of minutes later, I came back to it, wondering if it had completed but it had not. The system was actually compiling the new patch, which on a 512MB or 1GB droplet takes forever, lemme tell ya.  But we gotta do what we gotta do. In the meantime I googled how to update the gemsets for our apps.

Luckily, I ran across Ray Hightower's blog (Hi Ray!) about [Upgrading Ruby with RVM][2], a great starting point for me. Little did I know that upgrading to a new patch/version would be a bit more difficult if there wasn't a quick 'n' easy binary to download.

Personally, I'm more familiar with rbenv (use it locally), and I know the 2 are alike when it comes to gemsets and handling them. I tried to upgrade my machine, but since there weren't binaries for it yet, I couldn't. No big deal, the app servers are at least getting updated and that's more important right now.

In the end, the semantic versioned Rubies weren't a problem, since changing the `.ruby-version` file causes a new gemset to be created, but the patch version Ruby (2.0.0) was a lot harder to deal with.

First I'll go over what I did wrong/incorrectly.

### All the WRONG things
After installing 2.0.0-p645 and following Ray's directions, I tried to run `rvm upgrade`. After all, that should make things pretty quick and the gems will transfer over! Sweet.

```bash
$ rvm upgrade 2.0.0
Are you sure you wish to upgrade from ruby-2.0.0-p645 to ruby-2.0.0-p598? (Y/n):
```
Oh. Ok. So you want to go backwards. No no I don't want that. 

```bash
$ rvm upgrade 2.0.0-p598 2.0.0-p645
Are you sure you wish to upgrade from ruby-2.0.0-p598 to ruby-2.0.0-p645? (Y/n):
```

{% icon fa-angle-double-up %} Level up +1. +5 if unfamiliar with RVM like me.

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[1]: https://www.ruby-lang.org/en/news/2015/04/13/ruby-openssl-hostname-matching-vulnerability/
[2]: http://rayhightower.com/blog/2013/05/16/upgrading-ruby-with-rvm/
[email]: mailto:risaonrails@gmail.com