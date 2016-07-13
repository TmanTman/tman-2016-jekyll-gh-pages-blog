---
layout: post
title: Scala's Parameter vs Variable Call-by-name and Call-by-value declaration
---

The call-by-name and call-by-value in Scala are used to implement certain features such as
"short-circuit" if-else expressions and lazy streams.

As an example, consider the following if statement. This statement should always return 10, without considering the value
of anyVal.

{% highlight scala %}
if (true || anyVal) 10 else 20
{% endhighlight %}



**Variable Declarations**

To declare values as call-by-value, use the keyword *val*. The value of a is immediately calculate and stored as 40.

{% highlight scala %}
val a = 20 + 20;
{% endhighlight %}

To declare values as call-by-name, use the keyword *def*. The value of a is only calculated when required.

{% highlight scala %}
def a = 20 + 20;
{% endhighlight %}

**Parameters**

Parameters in Scala is passed as call-by-value by default. firstParameter is immediately calculated as 40.

{% highlight scala %}
def a(firstParam: Int) = ...
a(20 + 20)
{% endhighlight %}

To declare the parameters as call-by-name. Now firstParam is only evaluated when it is required.

{% highlight scala %}
def a(firstParam: => Int) = ...
a(20 + 20)
{% endhighlight %}
