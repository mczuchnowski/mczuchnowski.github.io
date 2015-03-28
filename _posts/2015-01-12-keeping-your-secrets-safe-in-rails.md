---
layout: post
title: "Keeping your secrets safe in Rails"
tagline: "Using environment variables and .gitignore"
description: "At some point you will want to extend your Rails app by connecting it to some
external service. It might be a mailer, Amazon AWS account for storing assets or
a Facebook app allowing users to sign up using Omniauth. Whatever it is, in most
cases you will get some secret credentials in the form of meaningless strings
that will allow the app to get recognized by said services and connect
to them. The thing is, if you're using version control (and who doesn't?), your
credentials (secret keys, passwords) will find their way to the hands of people
whose intentions are anything but ethical. So how do you inject these secret
strings into your Rails app without compromising security?"
category: tutorials
tags: [ruby on rails, rails 4, security, git]
---
{% include JB/setup %}

{{ page.description }}
<!--break-->

<h2>Environment variables</h2>

Remember: never EVER put your keys or passwords directly inside
your Rails app code. Even if you keep your code in a private repository, there
is no guarantee that this repository will always stay private. Things like this
should be kept in so-called environment variables. Make sure you read this
<a href="http://www.devfactor.net/2014/12/30/2375-amazon-mistake/">article</a>
to see what's at stake here.

You usually set environment variables manually through your server and refer to
them in your code using names that follow this pattern:
```ENV["VARIABLE_NAME"]```
. For example, Heroku allows you to set up environment variables through the
console or Heroku dashboard. But it's not that obvious in development and test
environments. Setting these up manually every time you run your local server or
test suite would drive you to madness, suicide or both.

There are many ways to set up your development and test environment variables.
The method I'll describe here is my preferred way of doing this which does not
depend on any external libraries or gems.

<h2>Your secret YAML file</h2>

I first read about this method on
<a href="http://blog.railsapps.org/">Rails Apps</a> blog. It involves 4 steps:

<h3>1) create the file for storing keys</h3>
You create a .yml file in your ```/config``` directory. I always name it
```local_env.yml```, but you can use any other name (just make sure you use the
same name in the next steps). Leave it empty for now.

<h3>2) add the file to .gitignore</h3>
This is the most important step - failing to follow it successfully will result
in compromising your credentials at some point. Make sure you add this line to
your ```.gitignore``` file (it's in the root of your Rails project folder):
<pre>/config/local_env.yml</pre>
Make sure your .yml file is in fact ignored by git. You can do this by running
```git status``` in your terminal and checking if it appears anywhere in the
output. Proceed to the next step only if you're sure that your .yml file is not
tracked by git!

<h3>3) enter your credentials</h3>
Now you can enter your secret keys inside the file. It follows the normal YAML
pattern, here's an example (no, these are not my real access keys):

{% highlight yaml %}
# supersecret gmail credentials
GMAIL_USERNAME: 'Gmail Man'
GMAIL_PASSWORD: 'mypassword123'

# supersecret amazon aws credentials
AWS_ACCESS_KEY_ID: 'AAABBBCCC111222333'
AWS_SECRET_ACCESS_KEY: 'aaabbbcccdddeee111222333///'
{% endhighlight %}

<h3>3) configure your application</h3>
The only thing left to do is configuring your application to check if your
secret YAML file exists and if it does - loading its contents as environment
variables. Add this somewhere inside your ```config/application.rb``` file,
inside the Application class (change the local_env.yml file name to whatever
name you're using):

{% highlight ruby %}
config.before_configuration do
  env_file = File.join(Rails.root, 'config', 'local_env.yml')
  YAML.load(File.open(env_file)).each do |key, value|
    ENV[key.to_s] = value
  end if File.exists?(env_file)
end
{% endhighlight %}

From now on you can refer to any environment variable set up in this file using
names like ```ENV["GMAIL_USERNAME"]```, ```ENV["AWS_ACCESS_KEY_ID"]``` etc. In
development and test environments these values will be taken from your secret
file and in production you have to set them up manually like you did before.

<h2>Conclusion</h2>
This is the very first entry on my blog because keeping your credentials out of
sight is crucial. I will be referring to this post on many occasions. Setting
this up is one of the first things I do whenever I generate a new Rails app.
There are gems that can make hiding credentials easier, but I don't fully trust
them in this regard. Stay safe!
