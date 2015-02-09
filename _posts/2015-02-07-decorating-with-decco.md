---
layout: post
title: Decorating with Decco
tags: [Ruby, Decco, Rails]
comments: true
---

I'll be honest, I'm not a fan of Rails helpers. There's something about poluting a global namespace with unorganized cruft that makes me twitch ever so slightly. How do we get away, or at least minimize the usage of helpers? Simple, we use a decorator. A decorator allows you to add additional functionality to an object without it deviating away from it's core responsibility. A quick search of Github yields many decorating systems, the most noteable being Draper. I've used Draper and think it's a fantastic library, but for me it feels a bit cumbersome for my needs. This is why I created Decco.

<a href="http://github.com/kris/decco" target="_blank" rel="nofollow">Decco</a> is a combination decorator/presenter system. It's designed to be lightweight and simple to use, built off simple Ruby classes. All you need to do is create a class that takes an object to be decorated when instantied. For example:

{% highlight ruby %}
class UserDecorator
  def initialize(user, view)
    @user = user
    @view = view
  end
  
  def gravatar_url(options = {})
    hash = Digest::MD5.hexdigest(@user.email)
    "http://www.gravatar.com/avatar/#{hash}?#{options.to_query}"
  end
end
{% endhighlight %}

From here you can instantiate a new instance:

{% highlight ruby %}
# Infers name from object -- UserDecorator
Decco.decorate(@user)

# Specify a decorator
Decco.decorate(@user, 'OtherUserDecorator')
{% endhighlight %}

If you're using this within Rails, then you get an additional helper for your views:

{% highlight ruby %}
# Get a decorator singleton instance
d(@object)
image_tag(d(@user).gravatar_url)
{% endhighlight %}

This helper instanties the decorator with the current view object so you can call Rails helpers within your decorator.

Ultimately, Decco strives to be as simplistic as possible. It's merely a simple wrapper to instantiate your decorator/presenter based on the object as well as provide a simple helper for caching the decorate call.

<a href="http://github.com/kris/decco" target="_blank" rel="nofollow">View Decco on Github</a>