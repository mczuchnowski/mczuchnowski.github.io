---
layout: post
title: "Activity feed with fine-grained control"
tagline: "Using public_activity gem in the controller"
description: "When you start using Public Activity gem, the documentation and most tutorials tell you to put the logic in the models that you want to track. This is fine if your app is not very complex and if you want to track all the actions related to the given model. But what if you need more control?"
category: tutorials
tags: [ruby on rails, rails 4]
---
{% include JB/setup %}

{{ page.description }}
<!--break-->

<h2>Initial Setup</h2>

This tutorial assumes that you are familiar with ```public_activity``` gem basics and you know what code goes where (using partials for each action, not the I18n variant).

When I first started using the gem, I followed the standard setup with ```owner``` defined in the tracked model:

{% highlight ruby %}
# app/models/status.rb
class Status < ActiveRecord::Base
  include PublicActivity::Model
  tracked owner: owner: Proc.new{ |controller, model| controller.current_user }
end
{% endhighlight %}

The ```Comment``` model also trached the associated ```Status``` as ```:recipient```. The above code also uses the hacky solution that enables ```current_user method to be used in the model, which is not veyr pretty.

This required me to create partials for every single method in the Status controller. The partials were named by default based on the names of the methods. And by default, they fired activity feed actions with the same names. But I required a more fine-grained control.

For starters, I didn't want all the methods to create activity feed objects - some of them were not very interesting and I didn't want them to litter the feed. But more importantly, I wanted some of the methods to fire custom-named activity feed actions under certain conditions. More specifically, with my upvoting and downvoting system, I wanted the activity to say "User cancelled their upvote" if the user clicked the upvote twice, which cancels the the upvote that was previously cast by the user for that particular status. Same for downvotes. With default setup, this would show two ```upvote``` activities in the feed, which isn't very informative, considering what really takes place under the hood. Plus, I didn't like the hacky solution for the ```current_user``` method. It caused some tests to fail too, although the app worked fine.

<h2>The Solution</h2>

Luckily, the Public Activity gem is very flexible and you can get more control over exactly when activities are created and how they are named.

The model now only has this line related to Public Activity:

{% highlight ruby %}
# app/models/status.rb
class Status < ActiveRecord::Base
  include PublicActivity::Common
end
{% endhighlight %}

With this in place, activities are not automatically created unless the proper method is called. For example, the create method in the status controller has this code:

{% highlight ruby %}
# app/controllers/statuses_controller.rb
...

if @status.save
  @status.create_activity :create, owner: current_user
else
  render action: 'new'
end

...
{% endhighlight %}

The ```current_user``` method used here is the classic method provided by Devise. No hacky solutions needed, no more weird test failures. But it gets more interesting with the custom ```upvote``` method:

{% highlight ruby %}
# app/controllers/statuses_controller.rb
...

def upvote
  if current_user.voted_up_on? @status
    @status.unliked_by current_user
    @status.create_activity :unupvote, owner: current_user
  else
    @status.upvote_by current_user
    @status.create_activity :upvote, owner: current_user
  end
end

...
{% endhighlight %}

As you can see, the ```upvote``` activity is fired up only if the user did not upvote that status before. If it's the second thime they clicked it, a custom ```unupvote``` activity is fired (I know, this isn't my best name for a method) and this activity has its own partial that renders a different message than the normal ```upvote``` activity.

And if I don't want some controller actions to create activities in the feed, I just don't use ```create_activity``` method inside their definitions. Plain and simple. Perfect for control freaks like myself.

<h2>Bonus Round: Devise</h2>

This is all candy canes and sugar plum fairies until you decide to add user-related actions to the feed when Devise is your tool of choice. Let's assume we just want an activity to be generated when a new user registers. This means we have to override the ```create``` action of the users controller. I already described the exact method in <a href="http://mczuchnowski.github.io/tutorials/2015/03/23/dynamic-flash-messages-in-devise/">this blog post</a>. In this case we want to override the registrations controller's ```create``` action by adding one line just under the ```if resource.persisted?``` condition:

{% highlight ruby %}
# app/controllers/registrations_controller.rb
class RegistrationsController < Devise::RegistrationsController
  def create
    build_resource(sign_up_params)

    resource.save
    yield resource if block_given?
    if resource.persisted?
      resource.create_activity :create
      if resource.active_for_authentication?
        set_flash_message :notice, :signed_up if is_flashing_format?
        sign_up(resource_name, resource)
        respond_with resource, location: after_sign_up_path_for(resource)
      else
        set_flash_message :notice, :"signed_up_but_#{resource.inactive_message}" if is_flashing_format?
        expire_data_after_sign_in!
        respond_with resource, location: after_inactive_sign_up_path_for(resource)
      end
    else
      clean_up_passwords resource
      set_minimum_password_length
      respond_with resource
    end
  end
end
{% endhighlight %}

Don't forget to specify the controller in the route:

{% highlight ruby %}
devise_for :users, controllers: { registrations: "registrations" }
{% endhighlight %}

The partial goes to a folder which corresponds to the Devise model you're using (usually it's ```user```). If you want to specify ```create_activity``` for more actions, you will have to override them as well.
