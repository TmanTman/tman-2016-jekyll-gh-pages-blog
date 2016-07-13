---
layout: post
title: Functional Programming and the substitution model
---

Scala theory includes the substitution model. What is it and what does it matter?

Imperative programming works with mutable values, that are mostly edited by side effects.
Functional programming does not allow re-assignment of variables, just as in math.

This rule in mathematics and functional programming holds:

{% highlight javascript %}
(a+b)x + (c + d) = ax + c + bx + d
{% endhighlight %}

Only because the following is not allowed

{% highlight javascript %}
a = 2;
(a+b)x + (c + d) = ax + c + bx + d
a = 3;
{% endhighlight %}

The substitution model implies that, just as in mathematics, once a function has
been defined, the values in the function can at any time be substituded with the real
values and the final answer will always be the same.

So, in short, a function can be written anywhere simply as its answer. So you can call the function
at anytime and as long as the input parameters are the same, the end result should be the same.
