---
layout: post
title: Setting default values in ActiveRecord
tags: [Ruby, ActiveRecord, Concerns]
comments: true
---

I constantly find a need for default values in my ActiveRecord models. Many people recommend database migrations for this, but unless it's for counters, I try to keep it application side. A common approach is to use an *after_initialize* call:

{% highlight ruby %}
class Account < ActiveRecord::Base
  # After initialization, set default values
  after_initialize :set_default_values
  
  def set_default_values
    # Only set if time_zone IS NOT set
    self.time_zone ||= Time.zone.to_s
  end
end
{% endhighlight %}

This is a standard ActiveRecord callback that gets called both when an object is instantiated and when retrieved from the database. Be sure to set the value conditionally, as you don't want to overwrite the value when it's pulled from the database.

I'm sure we can clean this up a little using <a href="http://api.rubyonrails.org/classes/ActiveSupport/Concern.html" target="_blank">ActiveSupport::Concern</a>. Concerns are very similar to standard Ruby modules, but with some added (and semi-controversial) functionality. All we need to do is add the following to *app/models/concerns/defaults.rb*:

{% highlight ruby %}
module Defaults
  # Added to instance of object
  included do
    after_initialize :apply_default_values
  end

  # Callback for setting default values
  def apply_default_values
    self.class.defaults.each do |attribute, param|
      next unless self.send(attribute).nil?
      value = param.respond_to?(:call) ? param.call(self) : param
      self[attribute] = value
    end
  end

  # Added to class of object
  class_methods do
    def default(attribute, value = nil, &block)
      defaults[attribute] = value
      # Allow the passing of blocks
      defaults[attribute] = block if block_given?
    end

    def defaults
      @defaults ||= {}
    end
  end
end
{% endhighlight %}

Including this file does the following:

1. Adds a *default* method for assigning mappings
2. Adds a *defaults* method for returning mappings
3. Defines a callback which iterates over the mappings and assigns the default values.

Using this concern is as simple including it into your model and calling *default* on the attribtues you want to have a default value.

{% highlight ruby %}
class Account < ActiveRecord::Base
  # Include the concern
  include Defaults
  
  # We can define here
  default :time_zone, Time.zone.to_s
  
  # Or pass a block
  default :time_zone do
    Time.zone.to_s
  end
end
{% endhighlight ruby %}

Not only is this simple to implement, utilizing concerns will DRY up your code. Just remember, if you're using a concern in only one model, there really isn't a reason for it.