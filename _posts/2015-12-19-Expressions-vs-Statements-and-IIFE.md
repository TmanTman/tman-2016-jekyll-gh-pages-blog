---
layout: post
title: Expressions vs Statemens and IIFE
---

All this information has been taken from http://benalman.com/news/2010/11/immediately-invoked-function-expression/

A statement is a control structure. Take for example this if statement:

{% highlight javascript %}
var a;
if (true) {
    a = 20;
} else {
    a = 30;
}
console.log(a); //20
{% endhighlight %}

An expression is evaluated to a value. Take this ternary expression:

{% highlight javascript %}
var a = (true) ? 20 : 30;
console.log(a); //20
{% endhighlight %}

Now, lets understand Immediately Invoked Function Expressions from this. Javascript reads the following function
statement and recognises it as a function declaration, after which it can be invoked by calling it with parentheses:

{% highlight javascript %}
function foo() {
    console.log('boom')
};
foo(); //'boom'
{% endhighlight %}

A javascript function can however not be immediately invoked, as shown in the following three examples. Different reasons for
failure is given after each example.

{% highlight javascript %}
function() {
    console.log('boom')
}();    //SyntaxError: Unexpected token (
{% endhighlight %}

This function fails because the Javascript parser sees a function declaration without a name. You have to explicitly tell the parser that it
is a function statement. More on how to do this later. The following two examples show that named functions also gives errors. The closing brackets after the name function is read apart from the function declaration and is interpreted as a grouping operator (i.e. brackets that control precendence).

{% highlight javascript %}
function foo() {
    console.log('boom')
}();    //SyntaxError: Unexpected token )
{% endhighlight %}

Notice that this time the parser gives an error and reports that the *closing bracket* is unexpected, where previously the *opening bracket*
was unexpected. The parser expects a grouping operator to have an expression inside.

{% highlight javascript %}
function foo() {
    console.log('boom')
}(1);
{% endhighlight %}

No error is thrown but the function is not invoked. This is because the parser interprets the code in the following way:

{% highlight javascript %}
function foo() {
    console.log('boom')
}
(1);
{% endhighlight %}

How is the parser informed that a function expression is intended and not a function declaration? A function declaration is a statement, not an expression. In Javascript, parentheses cannot contain a statement. If a function is placed in parentheses, it is evaluated as an expression. Two methods of achieving this is valid:

{% highlight javascript %}
(function() { ... })(); //recommended
(function() ) { ... };
{% endhighlight %}

Other cases where the parser already expects an expression does not require the use of parentheses:
{% highlight javascript %}
var i = function() { ... }();
true && function() { ... }();
{% endhighlight %}

Although the parser does read these as expressions, it's a very good idea to still write the parentheses so that developers reading the code can recognise that the function is immediately invoked.
