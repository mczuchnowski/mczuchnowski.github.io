---
layout: post
title: "Spinning icons on form buttons"
tagline: "Easy tweak for better user experience"
category: quick tips
tags: [ruby on rails, rails 4, jquery]
---
{% include JB/setup %}

I'm sure you've seen it somewhere: you click a submit button in an online form
(maybe when paying with a credit card), the button instantly becomes disabled
and gets a spinning icon that suggests it's doing something behind the scenes.
It enhances your user experience and makes the site visually responsive. You may
not know that this feature ships with Rails and is very easy to implement.
<!--break-->

Make sure you have this line in your ```application.js``` file, just under the
standard jquery declaration:

{% highlight javascript %}
//= require jquery_ujs
{% endhighlight %}

I'm assuming here that you use
[Font Awesome](http://fortawesome.github.io/Font-Awesome/examples/) to get your
various icons, including the spinning ones. If not, the html code inserted in
the last code snippet will be different for you.

Now, in your form, make sure you use the ```f.button``` declaration instead of
```f.input```. An example taken from my own Rails app:

{% highlight erb %}
...

<%= f.button "Register", class: 'btn btn-default' %>

...
{% endhighlight %}

All we need to do now is add a data attribute and tell it what we want to show
on the button once it's clicked and disabled. You can use all the html tags and
goodies here. Change the code accordingly if you're using another source for
the icons:

{% highlight erb %}
...

<%= f.button "Register", class: 'btn btn-default', data: { disable_with: "<i class='fa fa-spinner fa-spin'></i> Registering..." } %>

...
{% endhighlight %}

And that's it! This will automatically disable the button on click, put a
spinning icon on it along with the custom text "Registering...". No JavaScript
code required on your part. Bear in mind that this trick only works with form
buttons. You can't use it this way on your other buttons that just link
somewhere.
