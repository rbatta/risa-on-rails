---
layout: post
title:  "Postgres Workshop Notes"
date:   2015-03-18
categories: 
- postgres
- presentations
- guide
tags:
- postgres
permalink: /postgres-workshop-notes/
summary: "I gave a Postgres Workshop for WWC Ruby Tuesdays. Here is the full notes from it."
comments: true
---
Hi there!

Missed the Postgres workshop? Forgot what I covered? This post was made just for you then! :)

Quick background on Postgres. It's a relational database that is awesome because it's highly extensible. For instance there is geospacial support (postGIS), key/value store (hstore), and with postgres 9.4, there's JSON support (more so than what they had previously). You could say it's the end-all-be-all for databases. At least that's what I'd like to see it as! Obviously postgres may not suit your needs but this knowledge is valuable and transferable. And if you were afraid of reading the docs before, hopefully after this you won't be.

### What to expect
I'll walk thru entering/exiting Postgres. Then a bit of navigation so you feel more comfortable with it. And then we're going to go straight into dealing with import/export (restore/backup). For the import/export section, I've a chump vanilla Rails app that will be used because there's a rake task in there that creates fake data. Hurray for the Faker gem!

***

### First thing's first
Let's make sure you can run Postgres.

On a Mac, you have a couple of options:

* Download the [postgresapp](http://postgresapp.com/). Double-click to open and run it. Boom, postgres is running. You'll see it as the cute lil elephant icon in your menu bar up top. If you quit the app, you quit the postgres server. 
* Follow this [super awesome guide](http://www.mikeball.us/blog/setting-up-postgres-with-homebrew/). This is what I used to set my machine up. I prefer this to Postgresapp, truthfully.

On *nix machines, use [the official documentation](http://www.postgresql.org/download/linux/).

On Windows, you can use [this guide](http://www.postgresql.org/download/windows/)

Sorry *nix and Windows users. I'm a Mac user so the entire thing is geared for this. Don't hate!

***

### Go in, go out
Once you've followed the instructions for installation and have postgres running, we can confirm this by running `psql` in Terminal.

    $ psql

Did you get an error saying `FATAL role "YOURNAME" does not exist` ? It's saying that the user account you're logged in as isn't a postgres user. We've got to create it. Use the command below and make sure to use the name that the role says doesn't exist. Say yes when they ask if the user (you) should be superuser. 

    $ createuser --interactive YOURNAME
    # Shall the new role be a superuser? (y/n) Y

Did you get an error saying `FATAL database "YOURNAME" does not exist`? Simple enough, we'll just create it. Use the following command to create that particular database. Make sure you use the name that it references.

    $ createdb YOURNAME

Ok, in we go. The postgres prompt you see should have your account name if you followed the above steps. Since my account name is risa, my postgres username and database are risa. The prompt for me looks like `risa=#`.

    $ psql
    risa=#

Great. Let's quit out of postgres. The command is `\q` (not the usual / one. It's the \ above the return button)

    risa=# \q
    $

It's always important to know how to exit any program. Otherwise you might be stuck in [vim foreverrrrr](https://twitter.com/iamdevloper/status/435555976687923200).

***

### Navigation
To find out where we're at currently, type `\conninfo`. 

    $ psql
    risa=# \conninfo
    # You are connected to database "risa" as user "risa" via socket in "/tmp" at port "5432".

To list the databases you've got in postgres, type `\l` (l for list)

    risa=# \l
    # you'll get an output that lists a bunch of databases, including template0 and template1.

_Note: template1 is what all new databases use when they get created. If you add an extension like `pg_stat_statement` into template1, all your subsequent databases will have that extension turned on automatically. AWESOME.  
Template0 is the ultimate empty database. If you accidentally mess up template1, then you can use template0 to fix it. 
I'll include directions on this maybe at the end or in another blog, but this wasn't part of the workshop._

To change databases, you will have to do `\c DATABASE_NAME`. Since I don't know what databases you've already got in your system, I'll use template1 as an example.

    risa=# \c template1
    template1=#

Notice that the prompt changed. The prompt always points to the current database you're in. Sweet.

Let's hop out and do some Rails things so we can look a bit deeper into postgres.

***

### Rails setup
Please clone this repo: [image blog](https://github.com/rbatta/img_blog). It's my vanilla rails app based on the [Michael Hartl tutorial](https://www.railstutorial.org/book)  
It uses ruby 2.0.0, so you may have to use rvm or rbenv to change ruby versions for this app.

Once cloned and ruby version is changed to 2.0.0, run the usual suspects:

{% highlight bash %}
$ bundle install
$ rake db:setup
{% endhighlight %}

With Rails 4.0 and below, `rake db:setup` created a dev database with migrations run __as well as an empty test database__.  In Rails 4.1+, `db:setup` no longer creates an empty test database. You have to manually run `RAILS_ENV=test rake db:setup`. Silly, I know.

So, what did that rake db:setup do? Let's find out.

    $ psql
    risa=# \l
    # you should see these 2 added into your database list
    # imgblog_dev       | risa  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
    # imgblog_test      | risa  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

Boom. Now, let's hop into the imgblog_dev database and check out the tables and stuff. BTW, you can press tab to autocomplete database names (and sometimes table names too). The command for displaying tables is `\dt`.

    risa=# \c imgblog_dev
    imgblog_dev=# \dt
    # you should see 3 tables. images and users are the tables
    # schema_migrations has the indexes

If you're curious as to how big your tables might be, you can also do `\dt+` which gives you sizes as well.

To see what's in the tables, you can then do your standard SQL query:

    imgblog_dev=# select * from users;
    # this will display any/all your users and their columns. 
    # right now it's empty so you just see column names.

If you just want to see the column names of the users table, you can do `\d users` instead.

Great. Let's hop back out and put some real data in to do some backups and restores. Assuming you're still in the root directory of the img_blog, run the rake command to create a bunch of fake data.

    imgblog_dev=# \q
    $ rake db:populate

Hop in and confirm that data was created. I'm going to append the database name to the `psql` statement to automatically go to that database.

    $ psql imgblog_dev
    imgblog_dev=# SELECT * FROM users;
    # you should see a bunch of output that might be cut off
    # scroll with the arrow keys
    # get the prompt back by pressing Q.

Let's count how many users and images were created. (Answer: 33 users, 66 images)

    imgblog_dev=# select count(*) from users;
    # you get back 33
    imgblog_dev=# select count(*) from images;
    # you get back 66
    imgblog_dev=# \q

Awesome. Great job so far!! Time to start the backup / restore process. I'll go through the commands and append more flags as I go along, explaining each one. By the end, you should have about 4 backup files created.

***

### Backup and Restore (Export and Import) 

#### Backup first
Backups are created with the `pg_dump` command, but in order for it to know what to do, you have to append flags (options) to the command and send the output to a file.  We're going to be taking backups of `imgblog_dev`.

    $ pg_dump -d imgblog_dev > ~/Desktop/dump1.sql

If I had to read it out loud, it is saying "Take a dump (haha) of database imgblog_dev and send it to the Desktop and call it dump1.sql." The `-d` flag stands for database. The `>` is to output the contents to a file in some location. In this case it's to the Desktop, as dump1.sql. Why `.sql` ? Truthfully, the extension doesn't matter. It can be `.txt`, `.asdf`, or whatever. But for sanity sake, `.sql`. You'll see why when you open up the file.

    $ subl ~/Desktop/dump1.sql

Assuming you've got sublime as your text editor with that shortcut, that'll work. Otherwise do what you normally do when opening up files. :) When you do, you'll see that it's got a bunch of SQL commands. If you look more closely, you'll see that they have some `CREATE TABLE` commands and then `COPY` commands for the actual data. You've taken a human-readable backup of the database.

What if all we want is the data? Imagine we're transfering the information to another database and it's already got tables set up because we've run migrations. What then? Let's set some more flags.

    $ pg_dump -d imgblog_dev -a -O > ~/Desktop/dump2.sql

Ok, so we've set 2 more flags: `-a` and `-O` (capital o, not zero). The `-a` flag stands for `--data-only`, so telling it to take only the data. The `-O` command is `--no-owner` which means remove all traces of ownership when getting this backup.  If you open up this file and compare it to dump1, here's the kind of thing you'll see.

    # On ~ line 34 of dump1.sql, you'll see the owner as yourself
    -- Name: images; Type: TABLE; Schema: public; Owner: risa; Tablespace: 
    
    # On ~ line 15 of dump2.sql, you don't see an owner listed
    -- Data for Name: images; Type: TABLE DATA; Schema: public; Owner: -

Quick visual check also comfirms that dump2 only has data listed. No reference to tables.

Imagine you're working on a production database, hosted on Digital Ocean at IP address 127.0.0.1 and port 5432. (Reality: 127.0.0.1 == localhost == your computer's own IP address) This theoretical database is like 2GB large (whoa), and you need to transfer it to your dev env. It's gonna be a pain to download all 2GB. Let's compress it down, like an archive file.

    $ pg_dump -h 127.0.0.1 -p 5432 -d imgblog_dev -a -O -Fc > ~/Desktop/dump3.dump

The flags to connect: `-h` is for host, usually an IP or a .com kind of address; `-p` is for port, default postgres port is 5432 but it can be set to anything.  The next new flag is the `-Fc` flag. I like to think of it as "format compressed" but in reality it is "format custom." There are other `-F` options, like `-Ft` and `-Fd` which will also archive your file, but `-Fc` is the one you want. Finally, notice that I changed the extension to `.dump`. Again, it's a sanity thing. Appending that lets me know that it's actually a compressed file that I'm dealing with.

If you open this file, you'll immediately notice how it is completely human-readable gibberish and gobbledigook. :) It's compressed after all. Postgres can understand it; we just know that the file is a lot smaller in size.

    $ pg_dump -h 127.0.0.1 -p 5432 -U risa -d imgblog_dev -Fc -a -O > ~/Desktop/dump4.dump

This combines pretty much all the flags I've covered thus far with the addition of `-U` which is user. In your case, make sure you put your postgres username in, since you won't have a risa user (unless your name is Risa as well. Hi!)

Congrats! You've gotten this far. One last stretch. Importing/restoring things back in!

#### Then Restore
In the real world, you may have times when databases get completely effed up. Hopefully you've got redundant databases or at least a scheduled backup system set up. When this happens, we need to sometimes restore the database to a previously known state. It also means there's a chance of some loss of data. (Having a failover plan is essential, btw.) Othertimes, maybe you're pulling in prod data into staging so you can manipulate real data and run some load testing. Whatever the case, you need to import some data from one database to another.

Let's get to it. First we're going to totally break things, because when I gave the workshop, it was live, and I broke things :) We are going to restore to the `imgblog_test` database. 

But, one important thing. __One Very Important Thing__ When doing pg_dump, you have the two paths: human-readable and archive/compressed formats. If you choose the human-readable path, you ___MUST___ use `psql`. If you choose the archive/compressed format, you ___MUST___ use pg_restore. Got that? In pseudocode it's:

    use pg_dump
      if use -Fc
        use pg_restore
      else
        use psql
      end

Now that we got that sorted, let's use the compressed dump to import/restore.

    $ pg_restore -d imgblog_test < ~/Desktop/dump3.dump

If you've been following very carefully, you will get a bunch of error output. Postgres did NOT like what just happened.  Here's the proof:

    $ pg_restore: [archiver (db)] could not execute query: ERROR:  relation "images" does not exist
    Command was: COPY images (id, img_url, img_name, description, user_id, created_at, updated_at, tags, gif, pictures) FROM stdin;

Key in the part where it says: `ERROR: relation "images" does not exist`. Why'd it say that?  Remember how I said that in Rails 4.0, `rake db:setup` creates an empty test database? Yeah.... Empty database has no tables. The import would not have worked because dump3.dump is a data-only and no-owner dump. It needs tables for data to be inserted.  Instead, let's run dump1. Remember, since dump1 is a human-readable file, it needs to be run as psql.

    $ psql imgblog_test -f ~/Desktop/dump1.sql

The output should be a list of SET's and CREATE's and COPY's. They will correspond to the SETs and CREATEs and COPYs in your dump1.sql file. Confirm that data has been transferred properly.

    $ psql imgblog_test
    imgblog_test=# \dt
    # confirm you have images, schema_migrations, and users
    imgblog_test=# select count(*) from users;
    # you should have 33 users.
    imgblog_test=# select count(*) from images;
    # you should have 66 images.
    imgblog_test=# \q

Fantastic job! Let's drop the database and confirm that it is no longer there. I am using a shortcut to display the list of databases without hopping into postgres. Then we'll set it up properly.

    $ RAILS_ENV=test rake db:drop
    $ psql -l
    # confirm that imgblog_test does not exist
    $ RAILS_ENV=test rake db:setup
    # this runs db:create, migrate, and seed

Ok, now let's use that data-only dump file (dump3). The output of that is below the command.

    $ pg_restore -d imgblog_test < ~/Desktop/dump3.dump
    # the output
    pg_restore: [archiver (db)] Error while PROCESSING TOC:
    pg_restore: [archiver (db)] Error from TOC entry 2284; 0    34203 TABLE DATA schema_migrations risa
    pg_restore: [archiver (db)] COPY failed for table "schema_    migrations": ERROR:  duplicate key value violates unique    constraint "unique_schema_migrations"
    DETAIL:  Key (version)=(20141122220745) already exists.
    CONTEXT:  COPY schema_migrations, line 1
    WARNING: errors ignored on restore: 1

Oh no! There's an ERROR!! But wait, it says "duplicate key value violates unique constraint" That's just postgres double checking your work basically. No need to worry about that. It's basically saying that it already existed in the database, but that's to be expected, since we ran the migrations and created the tables (and indexes).  You can confirm that data indeed was still transferred over properly.  
...I will let you figure that one out on your own. :)

OK, ONE LAST RESTORE! Imagine that you have to restore the database from your machine to another machine. Yup, you'll need those host and port (and user, but skipping here) flags.  Before we do that, let's reset the test database.

    $ RAILS_ENV=test rake db:reset
    $ pg_restore -h 127.0.0.1 -p 5432 -d imgblog_test < ~/Documents/dump4.dump

Again, same kind of errors, but hop in and check the data. Did it transfer over?  Awesome!

***

### Wrap up

Congratulations!! You've learned how to navigate about in postgres, use some SQL commands, and import and export data from postgres. You rock! Give yourself a high-5 :D

Now, after reading all of this, I hope you feel a bit more comfortable reading the documentation. Here are some super helpful links.

* [Postgres Docs: psql](http://www.postgresql.org/docs/9.4/static/app-psql.html)
* [Postgres Docs: pg_dump](http://www.postgresql.org/docs/9.4/static/app-pgdump.html)
* [Postgres Docs: pg_restore](http://www.postgresql.org/docs/9.4/static/app-pgrestore.html)

***

Questions? Comments? Suggestions? Please let me know. You can email me at [risaonrails {% icon fa-paper-plane-o %}][email]. No question is too dumb.

{% icon fa-angle-double-up %} Level up +10

[email]: mailto:risaonrails@gmail.com
