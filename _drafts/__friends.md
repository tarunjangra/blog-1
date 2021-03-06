---
layout: post
title: "Can Objects Be Friends?"
date: 2016-11-14
tags: oop java
place: Moscow, Russia
description: |
  Can you have encapsulation with only objects and no naked data?
  The question boils down to how objects can interact without data.
  They can be friends!
keywords:
  - encapsulation
  - encapsulation friends
  - encapsulation trusted objects
  - encapsulation meaning
  - encapsulation violation
---

As discussed before, proper encapsulation leads to a complete absence
of naked data. However, the question remains: How can objects
interact if they can't exchange data? Eventually we have to expose
some data in order to let other objects use it, right? Yes, that's true.
However, I guess I have a solution that keeps encapsulation in place
while allowing objects to interact.

<!--more-->

Say that this is our object:

{% highlight java %}
class Temperature {
  private int t;
  public String toString() {
    return String.format("%d C", this.t);
  }
}
{% endhighlight %}

It represents a temperature. The only behavior it exposes is printing
the temperature in Celsius. We don't want to expose `t`, because
that will lead to the "naked data" problem we discussed earlier. We want
to keep `t` secret, and that's a good desire.

Now, we want to have the ability to print temperature in Fahrenheit. The most
obvious approach would be to introduce another method, `toFahrenheitString()`,
or add a Boolean flag to the object, which will change the behavior
or method `toString()`, right? Either one of these solutions is better than adding
a method `getT()`, but neither one is perfect.

What if we create this decorator:

{% highlight java %}
class TempFahrenheit implements Temperature {
  private TempCelsius origin;
  public String toString() {
    return String.format(
      "%d F", this.origin.t * 1.8 + 32
    );
  }
}
{% endhighlight %}

It should work just great:

{% highlight java %}
Temperature t = new TempFahrenheit(
  new TempCelsius(35)
);
{% endhighlight %}

The only problem is that it won't compile in Java, because class
`TempFahrenheit` is not allowed to access `t` in class `TempCelsius`. If
we make `t` public, everybody will be able to read it directly, and we'll have
that "naked data" problem&mdash;a severe violation of encapsulation.

However, if we allow that access _only_ to one class, everything will be fine.
Something like this: (It won't work in Java; it's just a concept.)

{% highlight java %}
class TempCelsius {
  trust TempFahrenheit; // here!
  private int t;
  public String toString() {
    return String.format("%d C", this.t);
  }
}
{% endhighlight %}

Since this `trust` keyword is placed into the class that allows access,
we won't have the "naked data" problem&mdash;we will always know exactly which
objects posess knowledge about `t`. When we change
something about `t`, we know exactly where to update the code.

What do you think?
