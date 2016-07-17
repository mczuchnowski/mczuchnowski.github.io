---
layout: post
title: "Making Rails layout actually responsive"
---

You applied Bootstrap's cool, responsive div classes or wrote your own media
queries in your Rails website. You test your new layout by resizing the browser
window in various ways and everything seems to work fine. But then you open your
site on your tablet or mobile phone and it's no longer responsive. The whole
desktop version of your site is crammed into a tiny screen. 'What the hell?',
you might ask.

You might have banged your head on the wall for hours trying to find out why the
site doesn't want to be responsive on mobile devices. I know I did. But then I
looked at someone else's code and found one interesting line in it, which fixed
my problem right away.

<h2>Responsive Meta Tag</h2>

The fix is quick and simple. All you have to do is add a single meta tag to your
application's main layout file, right between the ```<head>``` tags:

{% highlight html %}
<meta name="viewport" content="width=device-width, initial-scale=1.0">
{% endhighlight %}

It does come with other possible options and settings, I guess the one above is
the most common and universal one. A newly generated Rails app does not include
this by default. It basically takes the device's actual width and renders the
page based on that value. You should only use it if your site is designed for
responsive layout. Otherwise it could potentially make more harm than good.
