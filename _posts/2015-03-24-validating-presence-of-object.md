---
layout: post
title: "Validating presence of object"
tagline: "Instead of validating presence of its id"
description: "When we implement an association between two models, the model on the
```belongs_to``` end of the association has an additional ```_id``` field that
usually corresponds to the name of the model at the other end. When we write
validations, in most cases we just require the ```_id``` field to be present and
don't really care what's in there. Sometimes we may add a numericality
validation for it if we're using a SQL database and its default IDs. But can we
do better? Sure we can."
category: quick tips
tags: [ruby on rails, ruby 4]
---
{% include JB/setup %}

{{ page.description }}
<!--break-->

<h2>Object, not ID</h2>

Let's say we have a simple blogging app. It has blogposts and comments. A
blogpost has many comments, comment belongs to a blogpost. And we are validating
the presence of the ```blog_post_id``` field in the comment validations. It's
very common and totally OK. The validation may look something like this:

{% highlight ruby %}
# app/models/comment.rb
validates :blog_post_id, presence: true
{% endhighlight %}

With the code in its current state, I can go into the console, create a blogpost
with ```:id = 1``` and then create a comment with ```:blog_post_id = 666```.
The database will accept this, since I gave SOME value to the blog_post_id
field other than ```nil```, but blogpost with id 666 does not exist and the
comment will probably not get displayed anywhere until we create a blogpost with
id 666. That is why, with such associations, we may want to choose to do this
kind of validation:

{% highlight ruby %}
# app/models/comment.rb
validates :blog_post, presence: true
{% endhighlight %}

This new validation doesn't only check if the ```blog_post_id``` is present - it
checks if a blogpost with this exact id exists before it saves the comment into
the database. We just removed 3 characters, and with that we gained confidence
that the comment will not be created for blogposts that do not exist.
