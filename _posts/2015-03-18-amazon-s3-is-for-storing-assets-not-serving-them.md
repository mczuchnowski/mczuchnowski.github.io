---
layout: post
title: "Amazon S3 is for storing assets"
tagline: "CloudFront is for serving them"
description: "This quick tip is the result of my recent discovery. 99% of websites I've seen
that use Amazon's services (AWS) for storing and serving assets (such as images)
use S3 buckets for everything - they upload assets there and then serve those
assets from the same bucket. Even big players. And it's not the most efficient
or even the correct way to do this."
category: quick tips
tags: [aws]
---
{% include JB/setup %}

{{ page.description }}
<!--break-->

<h2>S3</h2>

I know that this is more convenient - store assets in and serve them from the
same location. But consider how this works: you set up an S3 bucket and you
choose a location for it (the default one is somewhere in the US). From now on,
if you upload something there, if someone from Europe visits your site, they
have to wait for the assets to reach them from that location in the States.
Every time. If you set up your bucket in Europe, people from outside Europe will
have the same problem and wait longer for images and other assets. This lowers
your own server's load, but doesn't increase performance much.

<h2>CloudFront</h2>

The way it was intended: you set up CloudFront service to serve assets on your
website and connect it to your S3 bucket. Let's say your S3 bucket is located in
the US. When a user from Sweden visits your website, CloudFront checks if assets
can be served from Amazon's server that is close to Sweden (it may even be
located in Sweden). If not, it will fetch the assets from the S3 bucket. So the
waiting time is the same with this first request from Sweden. But when this
visitor (or someone else from the neighborhood) visits your website again,
the assets are served much faster, because they come from a cache in a nearby
server.

It's not the easiest thing to configure and perhaps you've made an informed
decision to serve your assets from S3 bucket with good reasons for it. I just
want to make sure you know how it should be done if you want to provide the best
performance and responsiveness. The difference can be significant.
