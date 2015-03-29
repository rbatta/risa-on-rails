---
layout: post
title:  "A Quick Tip with Database Formatting"
date:   2015-03-20
tags: [postgres, mysql, tips]
categories: postgres mysql
permalink: /a-quick-tip-with-database-formatting/
---

I'm constantly learning, mostly at this new job. It's gotten to the point where I end up diving down the rabbit hole many many times. Don't get me wrong, I love every bit of it. Might even say I'm addicted to it. Today's tip is brought to you by both MySQL && Postgres! Whoa, a double feature.

### The problem
Know what I really don't like? Having to check the database and getting results that are displayed all sorts of wonky because your terminal window wraps things when you run `select * from that_table where id='1';`. It looks something like this in MySQL:

![MySQL]({{ site.baseurl }}/assets/quick-tip-mysql.png)

Oh man. Postgres isn't that much better, y'know.

![Postgres]({{ site.baseurl }}/assets/quick-tip-postgres.png)


### The Solution
So what do? In MySQL, every SQL command that needs to be formatted pretty needs to have \G appended at the end. `select * from that_table where id='1' \G;` Annoying to have to append `\G` to the end, but it's worth it. Here's the result:

![MySQL]({{ site.baseurl }}/assets/quick-tip-mysql1.png)

Postgres is a bit better in this regard. There's a global setting called `\x auto` which automatically prettifies the information based on terminal width. The only downside is if you've got a text column that's so long it makes the actual header wrap. I'll take it anyway, because once it's set, it's set until you change it.

![Postgres]({{ site.baseurl }}/assets/quick-tip-postgres1.png)

Right on! This makes viewing records directly in the database so much easier. 

{% icon fa-angle-double-up %} Level up +1

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com