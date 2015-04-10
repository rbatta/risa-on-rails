---
title: "Jekyll: The blog maker"
layout: post
date: 2015-03-31   
tags: 
- jekyll
- guide
categories: 
- jekyll
- guide
levelup: 5
permalink: /:title/
summary: "My guide to setting up a Jekyll blog on Github Pages"
comments: true
---
As you may have figured out, I use Jekyll to create my Github Pages. And boy did I have a hard time understanding what the heck was going on. I dunno, maybe docs and I just don't get along. Most of the how-to guides out there were like "oh yeah rip off this template from this site and start using jekyll!" but that feels like a cop-out to me, and I like to bash my head into the wall. Or keyboard.

![Bash your head in](http://i1.kym-cdn.com/photos/images/masonry/000/021/531/SmashingHeadOnKeyboard.gif)

Pretty. Much. Every. Day.

Anyway, here's what I've learned thus far with using Jekyll and Github Pages. Maybe this will help you out, too. I hope to expand my knowledge as well.  Glad that it's written in Ruby, so implementation of things are much easier to understand (for me).

### > Github Pages Overview
First issue I did not realize, and maybe I should have actually _read_ the instructions better,... there is a HUGE difference between creating a Github Page for a repo and for your account. I wanted to create one for my account but made it for my repo instead. Whoops.

Github [explains this fairly well](https://pages.github.com/), but here's my version for one-stop shopping goodness. I assume you're a developer and are knowledgable about the standard `git` type of commands.

On your local machine, install Jekyll.

{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

### > Github Pages for your github account
Create a new repo in Github called "your-github-username.github.io". In my case it's a repo called "rbatta.github.io" since my Github username is rbatta. When you git clone to your local machine, your `master` branch will be the one Github uses to publish your site. You get that? _Master branch is what Github uses to publish your site._ Git clone your new repo and hop into that directory.

_Note: If you create a repo and name it whatever you want and NOT "your-username.github.io", Github will see this as a Page for your Repo. When this happens, Github will use the branch called `gh-pages` instead of `master` to auto-publish your site. It's confusing and tripped me up massively in the beginning._

{% highlight bash %}
$ git clone git@github.com/your-github-username/your-github-username.github.io.git
{% endhighlight %}

For sanity's sake, before you hop into your directory, create a new directory and call it whatever you want. If it makes it easier for you, call it what you want your site to be called. Is it your blog? Call it that. Since I booched the whole Github Pages account vs repo thing, I already had a repo called `risa-on-rails` so I used this. This new directory thing will come in handy later. Trust me.

{% highlight bash %}
$ mkdir your-sites-blog-name
$ cd your-sites-blog-name
{% endhighlight %}

Great! Now I realize this is an empty directory. It's cool. Time to create a new Jekyll blog.

__But wait!__ Why are we in `your-sites-blog-name` and not in `your-username.github.io` directory? Well, like I said before, the `master` branch is what Github uses to publish your blog. ___NOT___ publish your repo. We'll keep the `master` directory clean until we're ready to push things up. So in the meantime, we'll use the `your-sites-blog-name` directory to do all of our work.

### > Jekyll setup
Create a new Jekyll site by running `jekyll new .` Make sure you've got that period in there, since that creates it within this directory.

{% highlight bash %}$ jekyll new .{% endhighlight %}

Jekyll will install a bunch of directories with _'s in them along with some other files, including a `_config.yml` file. This file will be the driver of just about everything: name it here and use it everywhere!

Create 2 directories called `_plugins` and `_drafts`. Because you may want/need them.

{% highlight bash %}$ mkdir _plugins _drafts{% endhighlight %}

This is probably a good time to do `git init` and do your initial commit.

### > _config.yml
Fill in the information given. For `baseurl:` keep that as blank quotes so that when you serve up your site, it'll load at `127.0.0.1:4000` instead of `127.0.0.1:4000/baseurl-you-gave-it`.

If you change any of the variables, make sure you change them in the css and anywhere else in the other files. For me, I changed the `:title` symbol to `:name` for the name of my blog. Anywhere where `site.title` was referenced, I changed to `site.name`. That includes the css file too. I did this because I added in a `permalink: /:title/` option to give all my pages their own permalinks instead of the weird directory structure Jekyll uses.

{% highlight yaml %}permalink: /:title/{% endhighlight %}

### > _posts/ and _drafts/
By default, anything that you put in the _posts directory will be published. Of course you need to make sure it's in the format of `yyyy-mm-dd-post-name` with whatever extension you use. I prefer markdown so all my posts end with `.md`. You're the author of your blogposts so choose whatever format you'd like: Textile or Markdown are the two default choices Jekyll supports. You can choose others with plugins.

The _drafts folder is exactly that, for drafts. You don't need to append dates to it. In fact, when you're ready to publish it, that's when you can append dates. For that, we've got a handy tool, the [jekyll-rake-boilerplate Rakefile](https://github.com/gummesson/jekyll-rake-boilerplate). Copy that Rakefile into your repo. Now you've got some added jekyll capabilities like: `rake watch["drafts"]` which is the equivalent to `jekyll serve --watch --drafts`, and `rake draft["your post's title"]` which creates a new draft post with the given title. Sweetness. `rake publish` let's you pick the draft you want to publish and appends the date automagically to your post, moving it from _drafts to _posts and eliminating the need to wonder what day it currently is.

To use the `rake draft["title"]` task, create a file called `_post.md` in the root directory. In your `_config.yml` add in the following so your rake task won't break when it's run:

    post:
      template: _post.md
      extension: md

In your `_post.md` file, you need to ensure that you've got the YAML markup at the beginning with the --- thing. If you need a template, copy paste it from the Rakefile site or use the welcome-to-jekyll post's YAML header. Think of this as your template for ALL your posts. Update as you see fit.

### > _plugins 
There are some cool plugins for added functionality for Jekyll. However when publishing to Github, that's stripped away and is not built when the site is published. Something something safety precautions. Ehhh. This is why we created a separate directory to work in. The `your-sites-blog-name` directory has all your drafts and posts and plugins and directories, but we will be creating the 'static' website and pushing that to the `your-username.github.io` directory for the master branch, to deploy to Github to publish. Wow, that came off a bit more complicated than I expected. Here's the workflow.  We'll go more into detail later on with this.

Start in your-sites-blog-name dir {% icon fa-angle-right %} create draft posts and stuff {% icon fa-angle-right %} rake publish draft {% icon fa-angle-right %} jekyll build site {% icon fa-angle-right %} copy _ONLY_ the `_site` directory contents to your-username.github.io directory {% icon fa-angle-right %} cleanup anything you need and git add, commit {% icon fa-angle-right %} push to github {% icon fa-angle-right %} ??? {% icon fa-angle-right %} profit!

Let's start by adding in my favourite simple plugin, the [jekyll-font-awesome](https://gist.github.com/23maverick23/8532525) plugin. Create a file in `_plugins/` called `font_awesome.rb` and paste the contents from that gist into this file. If you're not familiar with fontawesome.io, it's a bunch of rad icons for everything. Paste the link below into your head.html file at around line 6: `_includes/head.html`, just like that.

      <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">

Now you can add icons by using the syntax `{.% icon fa-something %}` (without the period in there). Go buck-wild and add some icons and stuff to your site! Wahoo!

### > Publishing
When you are ready to publish, build the site with Jekyll by running `jekyll build`.  This creates a directory called `_site/` which houses the raw ol' school HTML of your posts with a css directory that the pages reference.

OK. Remember the workflow up above? Let's break it down into simpler steps. ~~I ended up using Automator on the Mac (it's a native/default app and totally underrated IMO) to do this for me. The directions for that are beautifully written up [in this blog post](http://vincentp.me/blog/a-smarter-jekyll-workflow/).~~ *Edit: Automator let me down in the end. :cry:* Here is my updated workflow:

{% highlight bash %}
# starting in your-sites-blog-name directory
$ rake publish
# choose the draft post you want to publish
# that moves down to _posts
# make final checks and if all good then...
$ jekyll build
$ git add .
$ git commit -m "yeah your awesome message about your post"
$ sudo cp -r _site/* ../your-username.github.io/
# put in your password
# this overrides everything in the other directory
$ cd ../your-username.github.io
# confirm that the contents of _site were copied over
# you may need to rm -rf Rakefile OR add a .gitignore file
# and put Rakefile in it. Whatever you'd like to do
$ rm -rf Rakefile
$ git add .
$ git commit -m "your msg about your post that's ready for Github"
$ git push origin master
{% endhighlight %}

BOOM. You should be good to go now. Check out your site at http://your-username.github.io :)

### > Laziness
I like to make my workflow a lot easier by making aliases. The order of some of the commands won't matter. So here's the alias I created, using `sudo cp` to overwrite things properly. It involves putting in my password but that's still less typing overall. Easy peasy. This alias will be put into your `~/.bash_profile` or `~/.zshrc` or `~/.bashrc` file.  

{% highlight bash %}
alias copy-it='jekyll build;sudo cp -r _site/* ../your-username.github.io/;cd ../your-username.github.io;'
echo '======================='
echo 'Jekyll cheats'
echo '======================='
echo 'make sure you are in correct directory before running this'
echo 'copy-it = jekyll build then copy contents of _site to your-username master'
{% endhighlight %}

Obviously change the directory names to fit your thing. Save it and source your file: `source ~/.bash_profile` or `source ~/.zshrc` or `source ~/.bashrc`. You should see the commands displayed in your terminal.  Alternatively you can just restart terminal. 

_Note: If you run `copy-it` you *MUST* be in the correct directory (your-sites-blog-name)!! Don't forget to git add and commit prior to running `copy-it`, too!_

After running the command, you end up in your-username.github.io directory. Confirm your changes with a `git diff`. Then commit and push that sucker up! You are on your way to blogging freedom now. (And so am I!)

{% icon fa-angle-double-up %} Level up +5

***

Questions? Comments? Hit me up at [risaonrails {% icon fa-paper-plane-o %}][email]!

[email]: mailto:risaonrails@gmail.com