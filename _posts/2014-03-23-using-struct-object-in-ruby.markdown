---
layout: post
title: "Using Struct object in ruby"
categories: [ruby]
---

Ruby has a quite interesting concept in the Struct class. It's behavior is similar to a Hash object, but it includes additionally getter/setter methods for a set of predefined keys.

Basic exmaple:

{% highlight ruby %}

# Define a User Struct class with two fields, name and surname
User = Struct.new(:name, :surname)

# Create an instance
user = User.new

# name setter
user.name = "foo" # same as user[:name] = "foo"

# surname setter
user.surname = "bar"  # same as user[:surname] = "bar"

{% endhighlight %}

My first approach was:

EVENTS = [:new, :hang]
blocks = Struct.new(*EVENTS).new([], [])

This creates =>
blocks.new # same as blocks[:new])
blocks.hang # same as blocks[:hang])

The problem is that if you change the EVENTS constant and add a new supported event, you need to include a third parameter.

So I realized I needed to read a bit, after that I switched to:

blocks = Struct.new(*EVENTS).new(*Array.new(EVENTS.length, []))

It creates a new instance of the created Struct class with two elements initialized with an empty array, but it didn't work because each element is initialized with the same object

blocks.new.eql? blocks.hang => true

So the right solution is:

blocks = Struct.new(*EVENTS).new(*Array.new(EVENTS.length){Array.new()})

It creates a new Struct.new(*EVENTS) instance with the two elements (:new and :hang) initialized with an empty array instance (different instances).

{% highlight ruby %}

UserInfo = Struct.new("UserInfo", :name, :surname, :age, :sex) do
  def info
    Hash[members.zip(values)]
  end
end

u = UserInfo.new("Juan", "de Bravo", 35, "male")

# Get user name
p u.name
p u['name']

# Get User info
p u.info

{% endhighlight %}
