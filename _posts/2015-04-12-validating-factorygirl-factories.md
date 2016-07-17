---
layout: post
title: "Validating FactoryGirl factories"
---

It is very common for Rails developers to write dedicated tests that make sure that the given model has a valid factory associated with it. It seems like a good practice - these tests check if the factories used in other tests are set up correctly. If something changes in the validations, they will let you know where changes should be made. But, as always, we can do better and save some keystrokes.

<h2>What we used to do</h2>

Typical test for factory validity looks something like this:

{% highlight ruby %}
# spec/models/comment_spec.rb
...
it "has a valid factory" do
  expect(FactoryGirl.build(:status)).to be_valid
end
...
{% endhighlight %}

This builds a ```Status``` model object through FactoryGirl and passes it through the ```valid?``` method. But if you have lots and lots of factories, these tests add up and there is a much simpler and more efficient way to do this.

<h2>What we could do</h2>

I assume you're using RSpec, Factory Girl and Database Cleaner - a popular setup in Prime Stack. Create a file in ```spec/support/``` directory. It should contain the following code:

{% highlight ruby %}
# spec/support/factory_girl.rb

RSpec.configure do |config|
  config.before(:suite) do
    begin
      DatabaseCleaner.start
      FactoryGirl.lint
    ensure
      DatabaseCleaner.clean
    end
  end
end
{% endhighlight %}

(Database Cleaner is used here to make sure no artifacts are left after building factories with associations, which may happen on occasion)

What does this code do? Every time you run the ```rspec``` command for your tests, it will first check the validity of all the factories. If any invalid factories are found, the test suite will stop. All the necessary information will be output to your console.

<pre>
The following factories are invalid: (FactoryGirl::InvalidFactoryError)

* comment - Validation failed: User that actually exists must be assigned!
* status - Validation failed: User that actually exists must be assigned!
</pre>

In this actual example above I forgot to add associations to my factories. These are custom error messages that I set up for validations that check if the associated object actually exists, as described in
[this blog post]({% post_url 2015-03-24-validating-presence-of-object %}). The tests were working, because I always assigned proper associations before each test, but individual factories in isolation were not valid.

Now you can get rid of your "has a valid factory" tests altogether and just lint your factories automatically before each test suite run. Remember that from now on you will not be able to create factories for creating invalid objects (which you should not do anyway).
