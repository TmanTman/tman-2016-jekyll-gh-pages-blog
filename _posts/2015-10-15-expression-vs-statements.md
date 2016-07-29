---
layout: post
title: Expressions vs Statements
---

While programming my new Scryo app, I keep thinking that Functional Programming has some really good advantages
over OOP by not having side effects and mutable state.

So I rewatch the videos of Martin Oderskys Scala course from Coursera, and I understand some concepts that he mentions much better
than the first time I saw it.

He mentions that the If-else structure is not an statement but an expression. The difference is stipulated below.

Java uses if-else as an *statement*:

{% highlight java %}
boolean someBoolean = true;
int a;
if (someBoolean) {
    a = 10;
} else {
    a = 20;
}
{% endhighlight %}

Scala uses if-else as an *expression*:

{% highlight scala %}
boolean someBoolean = true;
a = if (someBoolean) 10 else 20
{% endhighlight %}

Notice that there are no side effect in the Scala code. The expression is evaluated
and returns a value.
