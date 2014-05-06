---
layout: post
title: "Using Struct object in ruby"
categories: [ruby]
---

# What is a Struct object

Ruby standard library includes the [**Struct**](http://ruby-doc.org/core-1.9.3/Struct.html) class. It's behavior is similar to a [**Hash**](http://ruby-doc.org/core-1.9.3/Hash.html) object, but it includes as well getter and setter methods for a set of predefined keys defined in the instantiation process.

This is the basic example for Struct usage:

{% highlight ruby %}

# Define a User Struct class with three fields, name, surname and email
User = Struct.new :name, :surname, :email

# Create an User instance
user = User.new

# Set the user name
user.name = "Juan" # same as user[:name] = "Juan"

# Set the user email
user.email = "juan@pollinimini.net"  # same as user[:email] = "juan@pollinimini.net"

{% endhighlight %}

Defining a Struct class is a simple way to have attributes validation in a lightweight object.

Find below three different ways to create an User Struct object (yes, Ruby zen does not state that *"There should be one -- and preferably only one -- obvious way to do it."*):

{% highlight ruby %}

# Way 1: one line defining a User class object
User = Struct.new :name, :surname, :email

# Way 2: "normal" class definition, inheriting from Struct and providing
# the User class attributes as parameter
class User < Struct.new :name, :surname, :email
end

# Way 3: Create the User class in the Struct namespace
Struct.new "User", :name, :surname, :email

user = Struct::User.new

{% endhighlight %}

# Beyond a simple usage

Let's consider the following exercise. You're working on a platform that can handle realtime communications, either text, voice or video. The idea of this exercise is to create a class to store a list of communication items that belong to a specific user. This could be easily done using the **Struct** class.

### Create a Struct instance using a predefined set of attributes

Taking advantage of the operator ***, we can use an Array as parameter to identify the The snippet below defines a class with an attribute for every communication type:

{% highlight ruby %}
# Predefined list of events to be considered
EVENTS = [:missed, :voicemail, :sms]
UserEvents = Struct.new(*EVENTS)
{% endhighlight %}

That's easy and quite straight forward, but if we want to define the class attributes as a stack of events, the initial value should be an empty array, not a nil value:

{%highlight ruby %}

user_events = UserEvents.new
user_events.missed
=> nil

{%endhighlight%}

And this is the awesome part! While creating an UserEvents instance, you can include the desired values to initialize the instance attributes:

{%highlight ruby %}

user_events = UserEvents.new(*Array.new(EVENTS.length){ Array.new() })

{%endhighlight %}

It creates a new UserEvents instance with the three elements (:missed, :voicemail and :sms) initialized with an empty array instance.

Bonus points, define a method to return the attributes names and values:

{% highlight ruby %}

UserEvents = Struct.new(*EVENTS) do
  def info
    Hash[members.zip(values)]
  end
end

user_events = UserEvents.new(*Array.new(EVENTS.length){Array.new()})

user_events.missed << :foo
user_events.sms << :bar

user_events.info
 => {:missed=>[:foo], :voicemail=>[], :sms=>[:bar]}
{% endhighlight %}
