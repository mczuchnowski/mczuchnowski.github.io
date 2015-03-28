---
layout: post
title: "Devise and Strong Parameters"
tagline: "Configuring custom user attributes"
description: "This blog post was originally intended for Treehouse students who try to do the
outdated Treebook course using Rails 4. It explains how to switch from Rails 3
model-based params whitelisting to Rails 4 controller-based params
whitelisting for mass assignment in the context of Devise which does not give
you an overt users controller to work with. It may also be helpful for anyone
trying to add some custom user parameters to a Devise-generated model and allow
users to actually use and update them."
category: tutorials
tags: [ruby on rails, rails 4, devise]
---
{% include JB/setup %}

{{ page.description }}
<!--break-->

<h2>attr_accessible</h2>

First off, let's talk about the difference between ```attr_accessor``` and
```attr_accessible```, because they look confusingly similar.
```attr_accessor``` is a Ruby method that creates a getter and a setter for your
class instance variables. You could define them on your own, but this makes it
much easier. ```attr_accessible``` is a Rails-specific method that, when used in
the model, allows mass-assignment of parameters of an object belonging to that
model.

Now, ```attr_accessor``` is independent from your Rails version, because it's a
Ruby thing, but ```attr_accessible``` was dropped in Rails 4 (using it will
cause and error) and in order to allow selected parameters to be mass-assigned,
you need to use the so-called Strong Parameters which are configured in the
controller. You should watch this
[Lynda video](http://www.lynda.com/Ruby-Rails-tutorials/Mass-assignment-strong-parameters/139989/159116-4.html)
to understand how mass assignment with Strong Parameters works.

<h2>Devise users controller</h2>

The problem with Devise is that you have direct access to the model, but you
don't see the users controller file anywhere. It's hidden somewhere inside the
gem. It's all fine if you want your users to only have the default parameters -
email and encrypted password. But what if you want to add more parameters, like
first_name, last_name or nationality? Sure, you can add them to the schema, but
they will not get assigned or updated through forms on your website. I will show
you two methods for allowing mass assignment of custom user parameters.

<h2>The lazy way</h2>

Devise documentation shows you a "lazy" method for adding custom parameters
to your user model and enabling them in mass assignments - they tell you what
you should add to the global application controller:

{% highlight ruby %}
# application_controller.rb
before_action :configure_permitted_parameters, if: :devise_controller?

protected

def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_up) << :username
end
{% endhighlight %}

Fine, but there are three problems with this. 1) you put devise-specific
functionality inside a global controller. It's easy (hence the word "lazy") and
maybe not that bad, but it's also not very elegant. 2) It only tells you how to
add one custom parameter. What if you want more? 3) It doesn't show you how to
allow these parameters to be updated. The above code only works for
registration (sign up) form.

The last two issues are easy to fix. The code below adds more custom
parameters and allows the user to also <strong>update</strong> them through
forms:

{% highlight ruby %}
# application_controller.rb
before_action :configure_permitted_parameters, if: :devise_controller?

protected

def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_up) << [:first_name, :last_name, :nationality]
  devise_parameter_sanitizer.for(:account_update) << [:first_name, :last_name, :nationality]
end
{% endhighlight %}

This will also work:

{% highlight ruby %}
# application_controller.rb
...

  devise_parameter_sanitizer.for(:sign_up) << :first_name << :last_name << :nationality
  devise_parameter_sanitizer.for(:account_update) << :first_name << :last_name << :nationality

...
{% endhighlight %}

You can fine-tune this and, for example, prevent users from changing their
first_name once they registered (just remove that parameter from the
```:account_update``` sanitizer line).

<h2>The elaborate way</h2>

This method requires an additional step, but in my opinion it's more
appropriate since it doesn't litter the global application controller. You
create a new initializer file
```/config/initializers/devise_permitted_parameters.rb``` and paste in the code
below:

{% highlight ruby %}
# config/initializers/devise_permitted_parameters.rb
module DevisePermittedParameters
  extend ActiveSupport::Concern

  included do
    before_filter :configure_permitted_parameters
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) << [:first_name, :last_name, :victim_relation]
    devise_parameter_sanitizer.for(:account_update) << [:first_name, :last_name, :victim_relation]
  end

end

DeviseController.send :include, DevisePermittedParameters
{% endhighlight %}

Change the custom parameters to fit your app and it will work the same way. The
initializer will get loaded when the application starts, allowing the users to
register with all the parameters and update them whenever they need to.

<h2>Conclusion</h2>

When you're switching from Rails 3 to Rails 4, Strong Parameters are crucial to
understand. Devise adds an extra layer of complexity by not giving you direct
access to the controller where you would want to set up additional parameters
for your Devise models. Hopefully, this post solved most of your issues.
