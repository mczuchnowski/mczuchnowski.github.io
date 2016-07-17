---
layout: post
title: "HTTP Basic Authentication in Rails"
---

Until a few days ago, the only Rails technique I knew for authentication and
authorization was through separate user model and session controller, with
registration forms, logging in, boolean ```admin``` fields, and with before
actions that check if the currently logged in user has proper permissions. This
was done manually or through gems like Devise or Warden (plus maybe CanCanCan or
Pundit). But what if that's too heavy and you really want something very basic
and simple?

Maybe you don't want to set up the whole user model with database migrations and
forms? Maybe you just want to restrict some parts of your website? Turns out
Rails has a feature for doing just that. And you rarely see it used in
tutorials, courses or books.

<h2>HTTP Basic Authentication</h2>

<strong>Warning: </strong>Before you start using HTTP Basic Authentication in
Rails, be advised that it's not the most secure method of authentication. Unless
you have an SSL certificate on your site, the credentials you enter when logging
in with this method are not securely encrypted. This might be an issue if, for
example, you're logging in while sitting in a café and using their public network
connection. Because of its simplicity, it's great for your private apps,
disposable training apps or pet projects, but you should not use it on
production sites that hold sensitive data without an SSL certificate.

<h2>Storing credentials</h2>

Before you start implementing basic authentication, you need a place to store
the credentials - username and password - that will allow access to restricted
parts of your site. You should not store them directly in your code. I suggest
using the method I described in my very first blog post:
[Keeping your secrets safe in Rails]({% post_url 2015-01-12-keeping-your-secrets-safe-in-rails %})

Throughout this tutorial I'll be assuming that you have a ```local_env.yml``` in
your app's ```config/``` directory, untracked by git, with content similar to
this (use any username and password, the ones below are just an example):

{% highlight yaml %}
# config/local_env.yml
ADMIN_NAME: 'admin'
ADMIN_PASSWORD: 'admin'
{% endhighlight %}

Whatever you put here, these are just going to be used in development and tests.

<h2>Implementing authentication</h2>

You most likely want to have an ```authenticate``` method which restricts access
to selected parts of your site and is available to more than one controller. You
may have a namespaced admin or dashboard controller, you can also use your
application controller. The method looks like this:

{% highlight ruby %}
# application_controller.rb or admin_controller.rb

protected

def authenticate
  authenticate_or_request_with_http_basic do |user, password|
    user == ENV["ADMIN_NAME"] && password == ENV["ADMIN_PASSWORD"]
  end
end
{% endhighlight %}

Notice that I'm referring to environment variables here. In development and test
environments these will be defined in your ```local_env.yml``` file (or whatever
other method you prefer), in production you will define them manually.

From now on you can use this before action for any controller actions that you
want to restrict to people who know the admin name and password:

{% highlight ruby %}
before_action :authenticate
{% endhighlight %}

When you visit the restricted part of the site, your browser will display a
window and ask you for the name and password. The appearance of said window will
vary depending on your browser and OS. If credentials are correct, you will not
be asked for them again throughout the restricted pages until you close and
reopen the browser.

<h2>Testing the authentication</h2>

Testing the HTTP Basic Authentication is almost as simple as using it in your
app. We are going to take advantage of environment variables here as well. As a
follower of the
[Prime stack](http://words.steveklabnik.com/rails-has-two-default-stacks) (and,
apparently, a victim of a
[cargo cult](http://www.rubyinside.com/dhh-offended-by-rspec-debate-4610.html)
as well), I will only show you the RSpec way of testing this:

{% highlight ruby %}
# admin or dashboard controller spec

before :each do
  request.env['HTTP_AUTHORIZATION'] =
    ActionController::HttpAuthentication::Basic.encode_credentials(ENV['ADMIN_NAME'],ENV['ADMIN_PASSWORD'])
end

describe "GET 'index'" do
  it "should be successful with proper credentials" do
    get 'index'
    expect(response).to be_success
  end
end
{% endhighlight %}

You can use this before block in any tests that deal with restricted part of the
app.

<h2>Conclusion</h2>

HTTP Basic Authentication is great if you need a lightweight way to restrict
access to some parts or actions on your website using just a few lines of code.
It is not very secure when used without an SSL certificate, so bear that in mind
before you implement it in a very sensitive project which you access through
public networks.
