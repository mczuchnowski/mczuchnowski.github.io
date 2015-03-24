---
layout: post
title: "Dynamic flash messages in Devise"
tagline: "Or how to override Devise controllers"
category: tutorials
tags: [ruby on rails, rails 4, devise]
---
{% include JB/setup %}

This blog post was inspired by a recent question on Treehouse forum. One of the
students asked how to create a dynamic flash message that the user would see
after signing in through Devise. I was sure that the ```devise.yml``` file's
messages can use basic Ruby string interpolation and that you could simply put
```#{current_user.email}``` inside the message. I was so wrong. I like
challenges, so after some searching and experimenting I managed to solve this
and now I would like to share what I've learned.
<!--break-->

<h2>What we want</h2>

Our goal is to get a customized welcome message after we sign in. Something like
"Welcome back, bob@email.com!", or whatever your email is, different for every
user that signs in. Right now we get the default flash notice that says "Signed
in successfully."

<h2>What Devise does by default</h2>

If you find the courage to visit GitHub and actually look inside Devise source
code (I know, I also hate discovering that it's not magic, but some more or less
complex Ruby code), you will see that by default, after signing in, Devise uses
its own .yml file to fetch flash message content. It all happens in this line:

{% highlight ruby %}
# devise/app/controllers/devise/sessions_controller.rb
...
set_flash_message(:notice, :signed_in) if is_flashing_format?
...
{% endhighlight %}

The thing is, we can't easily interpolate strings inside this .yml file, so the
best way to approach this would be to override the whole ```create``` method of
the Devise sessions controller and change the above line that sets the flash
message.

<h2>Setup</h2>

<strong>Note:</strong> while writing this tutorial, I was using Rails 4.2.0 and
Devise 3.4.1. It may not (and probably won't) work with different major versions
if you just copy and paste the code used in this blog post.

First, we need to create our own sessions controller that will inherit from the
default Devise sessions controller:

{% highlight ruby %}
# app/controllers/sessions_controller.rb
class SessionsController < Devise::SessionsController

end
{% endhighlight %}

We also need to change routing to tell Devise that we want it to use our custom
sessions controller for all its sessions-related stuff. This assumes that your
Devise model is called User:

{% highlight ruby %}
# app/controllers/sessions_controller.rb
devise_for :users, controllers: { sessions: 'sessions' }
{% endhighlight %}

With this, when looking for appropriate methods, Devise will first look at our
custom sessions controller and if if doesn't find the method it needs, it will
go deeper, to Devise's default sessions controller from which our own sessions
controller inherits. This is OOP 101.

<h2>Controller method</h2>

Now we go to Devise's GitHub repo and copy the whole ```create``` method from
the sessions controller into our new, custom sessions controller. Here, we can
change any behavior we want. And we want to change the flash message line that
I mentioned before. The new method will look something like this:

{% highlight ruby %}
# app/controllers/sessions_controller.rb
  def create
    self.resource = warden.authenticate!(auth_options)
    flash[:notice] = "Welcome back, #{current_user.email}" if is_flashing_format?
    sign_in(resource_name, resource)
    yield resource if block_given?
    respond_with resource, location: after_sign_in_path_for(resource)
  end
{% endhighlight %}

Now, whenever a user successfully signs in, they will see a flash notice which
will have their own email in it. If you customized your Devise model attributes
and have fields like ```name``` or ```first_name```, you can use them instead of
the ```.email``` above.

<h2>Conclusion</h2>

Looking into gem source code and changing its behavior is terrifying at first
and takes away the feeling of wonder that you have when starting with Rails and
gems like Devise. But at some point you have to learn how to do it and get used
to it to take full control over the app that you're writing.
