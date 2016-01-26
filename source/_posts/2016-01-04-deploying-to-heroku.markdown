---
layout: post
title: "Deploying Sinatra to Heroku, or how I learned to stop worrying and love postgres"
date: 2016-01-04 13:27:07 -0500
comments: true
categories: "Flatiron School"
---

Ahh, my first foray into placing something on the real webs. I hope that one day I look back on this as a fond learning experience, with much nostalgia on how much I had ahead of me. But to be real, the path to that day seems as fraught and horrible as my path to getting this goddamn site up. To keep it brief, I not only suffered, but tortured my dog with many violent outbursts of fervent cussing.

Where to begin?

Let's start with what Heroku is. The wikipedia definition is a cloud platform-as-a-service. In plain old english, it's a way to host and serve your code online. Heroku is lots of web servers (called dynos) which are capable of holding your code, running it, and serving it to people who want to use it. It supports lots of different languages and frameworks, and can even provide you with any nuts and bolts your app might need (like security or databases). Up to a certain point, it's free and a great way for us to deploy our websites in a professional, scalable manner.

##Heroku's Documentation

In short, the documentation is good. They do a great job of explaining, in a couple simple steps, how to get a rails app up and running with little to no effort.

Once you [sign up for a Heroku account](https://signup.heroku.com/signup/dc) and download the [toolkit](https://devcenter.heroku.com/articles/getting-started-with-ruby#set-up) you can upload an app with a few short command line requests:

`$ heroku login`

will bring up a field where you can enter the credentials for the account you created. Then, after navigating to repository of your app in your terminal, you can create a new app with the following command:

`$ heroku create my_awesome_app`

Now, all that's left to do is deploy your code. Luckily, heroku integrates seamlessly with git, so if you have a repository ready, you can push your entire codebase up with:

`$ git push heroku master`

If it seems to good to be true, that's because it is. The problem is, we are not using rails, we are using Sinatra - and on top of that, we're using the Tyrion Lanister of databases - sqlite3. It's totally functional, but it does not scale at all.

->![Tyrion](https://media4.giphy.com/media/Gqy93G8Ezezny/200.gif)<-

##Development Vs. Production

Working with Sinatra and Heroku requires a lot of rejiggering. First off, you're going to need a few more gems, and you are going to need to start to really demarcate the difference between your Development and Production environments. Right now, we've largely been working in what is considered Development environments.

An environment is the tier to which your code is deployed, or in short, where your code "lives". The definition of a development environment is pretty loose, but in general it is an environment where code can be changed freely. No thought has to be given to making changes in the development environment because it's usually isolated, or running on a single server or machine. In our case, our development environment is our computer.

However, now that we are considering serving our web application to other people, and hosting in on heroku where it can be accessed by just about anyone, we have to rethink what our code is. It is no longer this malleable hunk of clay that we can change at will. It needs to be stable, and a lot of thought needs to go into how and when it is changed. Heroku is a production environment, one where the code and application is publicly accessible, or actively serving it's purpose.

In big companies, there are generally steps in between Development and Production (such as testing, quality assurance and staging), but for now let's focus on these two. So what does this mean for us? It means we have to start thinking about what it means to push something up to production. It's time to get serious.

->![warning shot](https://cf.geekdo-images.com/images/pic323510.jpg)<-

##Getting with the Postgres
According to heroku, SQLite3 is not a production-grade database. Their reasoning as sound, and can be [read here](https://devcenter.heroku.com/articles/sqlite3) if you are interested.

This means we are going to have to upgrade our databases. Postgres is fairly easy to get your hands on, it's open source and readily available. You might already have it installed.

If you don't you can do so by typing in:

`brew install postgres`


Now, we need to make sure that your postgres db launches automatically. There is a good guide [here](http://www.tunnelsup.com/setting-up-postgres-on-mac-osx), which I'll summarize quickly.

After you've gotten postgres installed via brew, open it up in your finder and make sure it's running on your computer. Then you need to initialize your database, using the following command:

`initdb /usr/local/var/postgres`

You can continue following the instructions in order to get postgres to launch automatically, however, if you have it up and running right now, we can move on.

##Sinatra, you sweet, sweet lightweight fool

This is where Sinatra falls short of Rails. Rails takes care of a lot of the plumbing for you, but in Sinatra, we are going to have to do it ourselves.

After including the 'pg' gem in our gemfile, we are going to need to create a database.yml file in your config folder, right next to environment.rb.

->![Database.yml in action](http://i.imgur.com/TFZMqmw.png)<-

Here is a sample yaml file that serves as a decent template.

```yaml
default: &default
  adapter: postgresql
  pool: 5
  encoding: UTF8
  timeout: 5000
  port: 5432

development:
  <<: *default
  username: <%= ENV['PG_USER'] %>
  password: <%= ENV['PG_PASS'] %>
  database: db/my_db_development

test:
  <<: *default
  database: db/my_db_test

production:
  url: <%= ENV["DATABASE_URL"] %>
```
Two notes, the PG_USER and PG_PASS are environment variables that need to be set in your bash profile, thusly:

`export PG_USER = 'flatiron'`
`export PG_PASS = 'learnlovecode'`

You know, for safety reasons.

Second, the last line in this file is important for working with Heroku. Heroku will give your app an ENV (short for environment) hash with some data. In that hash is the database url, and will be essential for connecting the instance of a postgres db that heroku provides with your app.

Now we need to make sure that your gem file is divided into a development group, and includes the proper postgres gems. You will also need to include the 'pg' gem, which will act as your adapter between active record and postgres - much like our sqlite3 gem has done up until now.

Heroku REALLY hates sqlite. Make sure you delete the sqlite3 gem from any mention in your gemfile. Also, go into your Gemfile.lock and delete any mentions of sqlite3 to remove any accidental dependencies (or just delete your Gemfile.lock). Make sure to run bundle install before you try to redeploy.

```Ruby
source "https://rubygems.org"

ruby '2.2.3'
gem 'sinatra'
gem 'activerecord'
gem 'sinatra-activerecord'
gem 'sinatra-flash'
gem 'sinatra-redirect-with-flash'
gem 'rake'

#postgres!
gem 'postgresql'
gem 'pg'

group :development do
	gem 'pry'
	gem 'pry-nav'
  gem 'rspec'
	gem 'tux'
end
```

Next we move on to making the connection with active record. In your config/environment file, you will have to do some extra work to connect to postgres and make it compliant with Heroku. I took this piece of code from a [great guide for getting Sinatra to work with Postgres and Heroku](http://mherman.org/blog/2013/06/08/designing-with-class-sinatra-plus-postgresql-plus-heroku/#.VonyepMrJE5), so credit where credit is due.


```Ruby
require 'bundler'
Bundler.require(:default, :development, :production)

configure :production, :development do
  db =  URI.parse(ENV['DATABASE_URL'] || 'postgres://localhost/[the name you gave your db in your yaml file]')
#  db =  'postgres://localhost/topofthemorning_development'

  ActiveRecord::Base.establish_connection(
    :adapter  => db.scheme == 'postgres' ? 'postgresql' : db.scheme,
    :host     => db.host,
    :username => db.user,
    :password => db.password,
    :database => db.path[1..-1],
    :encoding => 'UTF8'

  )
end

require_relative '../app/controllers/application_controller.rb'
require_all 'app/models'

require "open-uri"
require "json"
```

The fifth line of code sets the database connection. It first checks to see if it's being served with a url (which we received from Heroku and assigned in our database.yml file), otherwise it uses the database we set up in our yml file.

As another note, make sure you name your local db on the other end of the || the same as you did in the database.yml file.

And that should do it! Your postgres db should be running locally just fine (really you should probably check). You can redeploy your app to heroku (with the `git push heroku master` command) and it should be up and running!

Keep in mind if your db has any data in it before, it will be gone now!

##Full Disclosure

I did not get this far on my own at all. Seriously, as of first writing this, my shit still didn't work. Thanks to help from some awesome people (thanks Sophie and Joe!), I've gotten it up and running.

However, that being said, I have a few pitfalls to avoid and few hints to give you so you can dodge the cesspit I found myself mired in for a few days.

1. When you make changes to your code, you need to push them up to git before you push them up to heroku. When you push to Heroku, it's coming from your git repository, not your local files.
2. Postgres is a lot stricter on the 'foreign key' constraint. If you write your migrations with belongs_to relationships, and define foreign keys, make sure that the table already exists in a previous migration. Sqlite3 will let that shit slide, but postgres is a lot less forgiving.
3. Pry doesn't work in your heroku! If you're code is broken it is much harder to fix once in a production environment. SO - a solid tip is make sure your code is working before you deploy it.
4. So Pry doesn't work, but you can still do some spot debugging by writing some rake tasks. For instance, if I want to see what's going on in my environment db, I can write some rake tasks to display the contents of a few of my models.
5. Rake tasks are an awesome way to interact with your code in general once it's in environment, having some tasks is a great way to do some maintenancy things, like clearing your db, or resetting a user, etc.
6. Make sure to set your public folder explicitly in your app controller! Otherwise, heroku will not serve your CSS, images, or any other files you link to your html from your public folder. Do so like this right in your application controller:
```Ruby
set :public_folder, File.join(root, "../../public")
```

That's all I got guys, hopefully, once you tune in next time this will be further down the road. You know what they say, when you shoot for the stars and miss, at least you still have cat gifs:

->![failcat](https://media0.giphy.com/media/jO161HOYUPKEM/200.gif)<-
