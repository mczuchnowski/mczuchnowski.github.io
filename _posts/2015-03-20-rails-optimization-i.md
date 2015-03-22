---
layout: post
title: "Rails Optimization I"
tagline: "Eager loading and counter cache"
category: tutorials
tags: [ruby on rails, optimization]
---
{% include JB/setup %}

Code optimization often sounds scary. But basic optimization in Rails doesn't
have to be scary or hard at all. As always, you have a great community to back
you up with gems that facilitate everything. In this first post about
optimization we will take a look at "eager loading", "counter cache" and
even take a quick peek at "unused eager loading". Bullet gem will be our guide.
<!--break-->

<h2>Introduction</h2>

For demonstration purposes, I will use a very basic application with common
features found in a lot of Rails tutorials - a very simple social networking
app, which you could probably even call an online forum. It's going to have
a User model, a Status model and a Comment model. Users will create statuses and
then leave comments (comments, therefore, will belong to users and statuses).
The index page will display the list of all statuses, author's name and the
number of comments for each status. This will let us see eager loading and
counter cache in action.

<h2>Setup</h2>

Bullet gem detects three very common types of issues that hurt the performance
of your Rails app: N+1 queries, unused eager loading and lack of counter cache.
First we have to add Bullet gem to our Gemfile. We just want in in the
development group.

{% highlight ruby %}
# Gemfile
group :development do
  ...
  gem 'bullet'
end
{% endhighlight %}

Then we have to configure Bullet's behavior. This is the code I usually use. It
gives you all the basic features and shouldn't raise any errors (I did have some
issues with Growl when using the default configuration posted in Bullet's
repository):

{% highlight ruby %}
# config/environments/development.rb
Rails.application.configure do

...

  config.after_initialize do
    Bullet.enable = true
    Bullet.alert = true
    Bullet.bullet_logger = true
    Bullet.console = true
    Bullet.rails_logger = true
  end

end
{% endhighlight %}

Here's the schema for our simple application:

{% highlight ruby %}
# db/schema.rb
create_table "comments", force: :cascade do |t|
  t.string   "body"
  t.integer  "user_id"
  t.integer  "status_id"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
end

create_table "statuses", force: :cascade do |t|
  t.string   "body"
  t.integer  "user_id"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
end

create_table "users", force: :cascade do |t|
  t.string   "name"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
end
{% endhighlight %}

And this is how our basic index view will look like. It's just the scaffolded
status index view with two additional columns - for status author's name and
status comments count:

{% highlight haml %}
// app/views/statuses/index.html.haml
%p#notice= notice
%h1 Listing Statuses
%table
  %thead
    %tr
      %th User
      %th Body
      %th Comments
      %th{:colspan => "3"}
  %tbody
    - @statuses.each do |status|
      %tr
        %td= status.user.name
        %td= status.body
        %td= status.comments.size
        %td= link_to 'Show', status
        %td= link_to 'Edit', edit_status_path(status)
        %td= link_to 'Destroy', status, method: :delete
%br/
= link_to 'New Status', new_status_path
{% endhighlight %}

With all this in place (and proper associations in the models), we can start
optimizing the application.

<h2>Eager loading</h2>

This is something we see very often - all statuses get loaded into this
```@statuses``` instance variable and then a new row is generated for each of
them. This is normally done in one database query which looks similar to this:

{% highlight sql %}
Status Load (0.2ms)  SELECT "statuses".* FROM "statuses"
{% endhighlight %}

But in our case, with two statuses in the database, when we visit the index
page, 3 queries are executed:

{% highlight sql %}
Status Load (0.2ms)  SELECT "statuses".* FROM "statuses"
User Load (0.3ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT 1  [["id", 1]]
User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT 1  [["id", 2]]
{% endhighlight %}

This is not very optimal. With every visit, we will be executing N+1 queries -
the total number of statuses plus one. Why is that? The culprit is our ```User```
column. If we want to display the names of the authors, for each status we have
to make an additional query to check the name of the user assigned to that
status. That is why Bullet will yell at us right away with a pop-up as well as
in the log when we open up our index page in the browser.

<pre>
N+1 Query detected
  Status => [:user]
  Add to your finder: :includes => [:user]
</pre>

Here, Bullet is telling us that we have an N+1 Query issue on that page. It's
also telling us exactly what we should do to fix it. We need to "eager load" the
user data along with the list of statuses. Right now, this is how our statuses
controller's index action looks like:

{% highlight ruby %}
def index
  @statuses = Status.all
end
{% endhighlight %}

All we need to do is add a little something to this finder:

{% highlight ruby %}
def index
  @statuses = Status.all.includes(:user)
end
{% endhighlight %}

From now on, no matter how many statuses and users we have in our database, this
view will only execute two queries, which is much more optimal and does not
cause Bullet to yell at us:

{% highlight sql %}
Status Load (0.2ms)  SELECT "statuses".* FROM "statuses"
User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (1, 2)
{% endhighlight %}

Well, OK, it still yells at us, but with a different issue.

<h2>Counter cache</h2>

Bullet describes our  current issue like this:

<pre>
Need Counter Cache
  Status => [:comments]
</pre>

And if we look closely at the log, with 4 statuses in the database, some
additional queries are being executed aside from the two that we saw earlier:

{% highlight sql %}
(0.2ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."status_id" = ?  [["status_id", 1]]
(0.1ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."status_id" = ?  [["status_id", 2]]
(0.1ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."status_id" = ?  [["status_id", 3]]
(0.1ms)  SELECT COUNT(*) FROM "comments" WHERE "comments"."status_id" = ?  [["status_id", 4]]
{% endhighlight %}

The culprit in this case is the ```status.comments.size``` part of our view.
For each status a query is made to see how many comments that status has. Every
single time. We often display such information, but we rarely think about the
overhead this produces. Rails has a feature to take care of that. It's called
"counter cache".

In order to implement counter cache properly, we first need to make a new
migration. We want each status to keep the number of its comments in a separate
column that updates itself every time we create or delete a comment. That column
has to follow a naming convention in order to be picked up as the counter cache
column. In our case, the migration will look like this:

{% highlight ruby %}
def change
  add_column :statuses, :comments_count, :integer, default: 0
end
{% endhighlight %}

But that's not all! This works fine if you just started working on the app and
it doesn't have anything in the database. But you probably already have some
statuses and comments, and you don't want the counters to show 0. We want the
migration to run additional code that will update comment counters for our
current statuses, so the code will look like this:

{% highlight ruby %}
def change
  add_column :statuses, :comments_count, :integer, default: 0
  Status.find_each do |status|
    status.update_attribute(:comments_count, status.comments.length)
  end
end
{% endhighlight %}

We are almost done. Only two short steps left. Now we need to update the Comment
model association so that it will increase or decrease the counter when we
create or destroy comments:

{% highlight ruby %}
# app/models/comment.rb
belongs_to :status, counter_cache: true
{% endhighlight %}

The last step is updating the index view to use the new counter column instead
of the ```.size``` method:

{% highlight haml %}
%tbody
  - @statuses.each do |status|
    %tr
      %td= status.user.name
      %td= status.body
      %td= status.comments_count
      %td= link_to 'Show', status
      %td= link_to 'Edit', edit_status_path(status)
      %td= link_to 'Destroy', status, method: :delete
{% endhighlight %}

Bullet no longer yells at us. No matter how many statuses, users and comments
we have in the database, only two queries are made to fetch all the data
required for this particular view.

<h2>Unused eager loading</h2>

If, for some reason, we remove the ```status.user.name``` column from the index
view, Bullet will yell at us once again, this time because we are eager loading
users along with the statuses and we're not doing anything with this data.

<pre>
Unused Eager Loading detected
  Status => [:user]
  Remove from your finder: :includes => [:user]
</pre>

This is Bullet's way of saying "hey, we can make even less database queries to
get the same effect!", which would translate into better performance. Just
remove the ```.includes(:user)``` part in your controller's index action finder
and it's all good.

<h2>Conclusion</h2>

Bullet is a brilliant gem that will detect the most common performance problems
in your app and even tell you what to do to fix them. This is helpful for
beginners and veterans alike - even if you're an experienced dev, you may forget
to do counter caching or eager loading somewhere in your app.
