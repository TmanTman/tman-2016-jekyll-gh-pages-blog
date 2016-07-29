---
layout: post
title: Clean Code -  Parameters or Arguments
---

When writing code, it should be written so that it's easy to read as well as easy to test.

To make code easy to test, it should require as little setup as possible (to establish the internal values or state required
by the function you wish to test). It's much easier to pass the values as parameters.

But removing unnecessary parameter calls can clean up your code. Take the following javascript code as an example.

Initial code:

{% highlight javascript %}
function Car(speed) {
  var a = speed;
  var speedUp(number) = {
    if (notOverSpeedLimit(number)) {
      increase(number);
    }
    function testIfCan(number) {
      return ((a + number) < 120);
    }
    function increase(number) {
      a = a + number;
    }
 }
}
{% endhighlight %}

To test the speedup function, you need to setup the object Car. The creator of Angular, Misko, advices us to not mix code that
instantiates objects and has business logic, so ideally it should be done through a factory.

It's easier to test a function where we can pass all the parameters, which gives the test

{% highlight javascript %}
speedUp(curSpeed, number)
{% endhighlight %}

expect(speedUp(20, 40)).toBe(60);

But it's also cleaner to not repeat values available in the scope to the internal method parameters. Take this implementation of Newton's squareroot method, taken from Martin Oderksys' Coursera course:

{% highlight javascript %}
sqrt(2);
function sqrt(findSqrtOf) {
    iter(findSqrtOf, 1);
    function iter(findSqrtOf, guess) {
        console.log('Iterating with values:', findSqrtOf, guess);
        var q = quotient(findSqrtOf, guess);
        if (isGoodEnough(q, guess)) {
            console.log('Found it:', q);
        } else {
            iter(findSqrtOf, q);
        }
    }
    function isGoodEnough(q, guess){
        var bool = (Math.abs(q - guess)/guess < 0.001);
        return bool;
    }
    function quotient(findSqrtOf, guess){
        return (findSqrtOf/guess + guess)/2;
    }
}
{% endhighlight %}

Can be refactored to :

{% highlight javascript %}
sqrt(2);
function sqrt(findSqrtOf) {
    iter(1);
    function iter(guess) {
        console.log('Iterating with values:', findSqrtOf, guess);
        var q = quotient(guess);
        if (isGoodEnough(q, guess)) {
            console.log('Found it:', q);
        } else {
            iter(q);
        }
    }
    function isGoodEnough(q, guess){
        var bool = (Math.abs(q - guess)/guess < 0.001);
        return bool;
    }
    function quotient(guess){
        return (findSqrtOf/guess + guess)/2;
    }
}
{% endhighlight %}
