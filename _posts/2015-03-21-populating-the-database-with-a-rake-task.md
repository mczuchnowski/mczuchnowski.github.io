---
layout: post
title: "Populating the database using a rake task"
---

There is a Rails convention for populating the database - through the
```db/seeds.rb``` file. But I sometimes want to just add some additional objects
to the database. For example, populate it with 100 entries, see how it performs
and then add 100 more. There is an easy trick for this that I've learned from
Hampton Catlin.

<h2>Custom Rake Task</h2>

First, if you want to have randomized, believable objects in your database, you
have to add the Faker gem to your Gemfile. I usually have it in my
```:test``` and ```:development``` groups:

{% highlight ruby %}
# Gemfile

gem 'faker'
{% endhighlight %}

Now you just have to create a new file, let's call it ```fake.rake``` and put
it in the ```lib/tasks/``` directory. Here's a very basic example of the
contents for that file:

{% highlight ruby %}
# lib/tasks/fake.rake

task :fake => :environment do
  puts 'Generating users...'

  30.times do
    User.create(
      name: Faker::Name.name,
      country: Faker::Address.country,
      city: Faker::Address.city)
  end

  puts 'done!'
end
{% endhighlight %}

Now, every time you run ```rake fake``` in the console while in your app's
folder, the database will be populated with 30 additional users whose names,
countries and cities of origin will look pretty convincing. And that's it! You
can later tweak the file by adding more tasks, namespacing the tasks or printing
the name of the environment in which the task is run.
