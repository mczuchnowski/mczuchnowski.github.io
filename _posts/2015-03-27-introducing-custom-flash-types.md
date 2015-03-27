---
layout: post
title: "Introducing custom flash types"
tagline: "For all your non-default message types"
category: quick tips
tags: [ruby on rails, rails 4]
---
{% include JB/setup %}

Out of the box, Rails only knows two types of flash messages: ```:notice``` and
```:alert```. You generally use the former for informational purposes and the
latter for errors and warnings. One of the questions on Treehouse forum reminded
me that you can use other types with custom names BUT they first have to be
defined.
<!--break-->

<h2>New types of messages</h2>

Developers usually want to introduce flash message types like ```:warning``` or
```:success``` and give such messages different colors in the layouts. This is
especially useful when working with CSS frameworks like Bootstrap or Foundation.

Since ```flash``` is basically a hash, all we need to do is add our custom types
to this hash. Rails 4 has a separate method for it which we put inside the
application controller (I'm assuming we want the new flash types to be
available to all controller that inherit from application controller).

{% highlight ruby %}
# app/controllers/application_controller.rb
add_flash_types :error, :danger, :success
{% endhighlight %}

With this line in place, we can use these new flash types in the controllers:

{% highlight ruby %}
# app/controllers/products_controller.rb
redirect_to products_path, error: "Something went wrong"
{% endhighlight %}

and display them in the views:

{% highlight erb %}
<%# app/views/products/index.html.erb %>
<%= error %>
{% endhighlight %}
