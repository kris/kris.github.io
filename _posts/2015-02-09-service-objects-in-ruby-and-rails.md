---
layout: post
title: Service objects in Ruby and Rails
tags: [Ruby, Rails]
comments: true
---

A common design pattern for performing tasks after an object is created is to use an <a href="http://api.rubyonrails.org/classes/ActiveModel/Callbacks.html" target="_blank" rel="nofollow">ActiveModel Callback</a>. For example:

{% highlight ruby %}
class User < ActiveRecord::Base
  after_create :send_welcome_email
  
  def send_welcome_email
    # Send an email
  end
end
{% endhighlight %}

Yes, this is simplistic, but there are a few problems with this.

1. It's not the User models responsibility to send an email.
2. Unless it modifies internal state, callbacks should be avoided.
3. Testing becomes painful and often times requires stubbing.

Lets talk about responsibility for a moment. In **my** opinion, if it's an interaction, it shouldn't belong to one specific model. What if you need a *send_invoice_email* to go with *send_welcome_email*? This can quickly get out of hand. This is why I use *service objects*.

So what exactly is a service object? It's really just an object that encapsulates operations. Using our initial callback example, lets refactor it to use a service object by adding the following to *app/services/send_welcome_email.rb*

{% highlight ruby %}
class SendWelcomeEmail
  def self.call(user)
    UserMailer.welcome_email(user).deliver
  end
end
{% endhighlight %}

Now to send a welcome email, you would do:

{% highlight ruby %}
SendWelcomeEmail.call(user)
{% endhighlight %}

This makes it far easier to test and decouples the responsibility.

### Implementations
If you've read other articles on service objects, you've probably run into multiple implementation methods. Some developers advocate that a service object should only respond to *call*, and only perform a single task. I don't see a reason for being so nitpicky. Instead, my service objects encapsulate related responsibility. For example, integrating with a third-party service:

{% highlight ruby %}
class StripeCustomer
  def initialize(member)
    @member = member
  end
  
  # Create a new stripe customer
  def create
  end
  
  # Update existing stripe customer
  def update
  end
  
  # Fetch the stripe customer info
  def fetch
  end
end
{% endhighlight %}

This is a much cleaner approach than, say, creating the following:

  * app/services/stripe/customer_create.rb
  * app/services/stripe/customer_update.rb

At the end of the day, simply separating this logic is going to make your life a lot easier.
