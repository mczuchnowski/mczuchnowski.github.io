---
layout: post
title: "Turbolinks and jQuery"
tagline: "How to make them work together?"
description: "Turbolinks were introduced in Rails to improve responsiveness. They make
your site much smoother, closer to a single-page application. However, they also
introduce a minor complication when you want to use jQuery (or any JS for that
matter). Imagine: you write your jQuery code, wrap it in the usual
```$(document).ready()```, run the server and the code seems to work fine.
But then you click some links here and there, provoke your jQuery
code to run again and...nothing happens. It doesn't work anymore. This can be
frustrating if you're just starting your learning adventure."
category: tutorials
tags: [ruby on rails, rails 4, jquery]
---
{% include JB/setup %}

{{ page.description }}
<!--break-->

<h2>Problem</h2>

Here's a sample code from a social app that I once made:

{% highlight javascript %}
// statuses.js
$(document).ready(
  $(".thums-up, .thumbs-down").click( function() {
    if ($(this).hasClass('highlighted')) {
      $(this).removeClass('highlighted');
    } else {
      $(this).siblings().removeClass('highlighted');
      $(this).addClass('highlighted');
    }
  });
);
{% endhighlight %}

It basically listens to click events and applies the ```.highlighted``` class to
the clicked thumbs-up/thumbs-down icon if it doesn't have that class yet
(removing that class from sibling objects at the same time) or removes that
class if the icon already has it. In this setup, if the Rails app uses
Turbolinks, this will only work when you first load the page or reload it using
F5/browser's refresh button. If you visit that same page through a link from
another place on the website, the clicking will no longer do anything.

It seems weird at first - your first reaction is "my JS code sucks". But it
worked before! Does that mean the document was not loaded and your page is not
ready? No. It means that another event got fired, but it wasn't your good ol'
friend ```(document).ready```.

<h2>Solution</h2>

After encountering that problem and doing a short search, I found a relevant
question and answer on Stack Exchange. Turbolinks fire another event called
```page:load```. So what you want to do is wrap your code in a function and run
that function whenever ```(document).ready``` <strong>or</strong>
```page:load``` are fired. In this app, I called my function ```ready``` and the
new code looks like this:

{% highlight javascript %}
// statuses.js
var ready;

ready = function() {
  $(".thums-up, .thumbs-down").click( function() {
    if ($(this).hasClass('highlighted')) {
      $(this).removeClass('highlighted');
    } else {
      $(this).siblings().removeClass('highlighted');
      $(this).addClass('highlighted');
    }
  });
};

$(document).ready(ready);
$(document).on('page:load', ready);
{% endhighlight %}

As you can see in the last two lines, the whole custom ```ready``` function runs
whenever the user visits the page, refreshes it or ends up on it by clicking a
turbolink somewhere on the website. The solution is easy, but not very obvious.

<h2>Bonus Round</h2>

Now, let's say that you expand your app further with some more AJAX to dodge
unnecessary page refreshes. Here's a sample code that, after creating a new
status, re-renders all the statuses on the page, slides the form up and shows
the "New Status" button again:

{% highlight javascript %}
// create.js.erb
$('#statuses').html("<%= j (render @statuses) %>");
$('#status-form').slideUp(350);
$('#status-maker').slideDown(250);
{% endhighlight %}

This causes a new problem: these re-rendered statuses were not there when you
fired your ```(document).ready``` or ```page:load```, so they will not react
to the cool ```ready``` function that adds and removes highlighting. Or the old
statuses will react, but not the new one that just got created. We need a way to
run our custom function also when new elements are dynamically added to the DOM.
It's quite simple, actually. We just need to name a new event, trigger it and
add it to the list that currently has our two events that already run our custom
```ready``` function. Let's call it ```statusesLoaded```. Here are the final
versions of both files:

{% highlight javascript %}
// create.js.erb
$('#statuses').html("<%= j (render @statuses) %>");
$('#status-form').slideUp(350);
$('#status-maker').slideDown(250);
$('#statuses').trigger("statusesLoaded");
{% endhighlight %}

{% highlight javascript %}
// statuses.js
var ready;

ready = function() {
  $(".thums-up, .thumbs-down").click( function() {
    if ($(this).hasClass('highlighted')) {
      $(this).removeClass('highlighted');
    } else {
      $(this).siblings().removeClass('highlighted');
      $(this).addClass('highlighted');
    }
  });
};

$(document).ready(ready);
$(document).on('page:load', ready);
$(document).on('statusesLoaded', ready);
{% endhighlight %}

Now the custom ```ready``` function runs whenever the user visits the page,
refreshes it, ends up on it by clicking a turbolink <strong>OR</strong> adds a
new status to the list dynamically.

<h2>Conclusion</h2>

JavaScript (with or without jQuery) is something you will have to deal with
rather sooner than later in your Rails development career, no matter how ugly
the language is and how many minds it destroyed (seriously, it's worse than
reading Alhazred's "Necronomicon", which is guaranteed to drive you mad). It's
good to know how to deal with this very basic and common problem.
