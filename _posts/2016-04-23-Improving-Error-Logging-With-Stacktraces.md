---
layout: post
title: Improving error logging with stacktraces
---

In the [previous post](http://localhost:4000/Basics-For-Remote-Logging/) I looked at
remote logging errors from a front-end application. The error messages for both the
global and Angular errors were not descriptive. The global errors were handled by
Logentries.

Enter the stage: [StacktraceJS](https://github.com/stacktracejs/stacktrace.js). The
goal is to get stacktraces from both the global Javascript window.onerror handler,
as well as from the Angular errors caught in the $exceptionHandler.

For the global window.onerror logging, the aim was to get better logging for the following exception:
{% highlight javascript %}
one();
function one() {
    two();
}
function two() {
    three();
}
function three() {
    var error = new Error('BOOM');
    throw error;
}
{% endhighlight %}

Without stacktracejs, the remote log shows:

{% highlight javascript %}
23 Apr 2016 09:21:21.709 196.215.130.135 s1p6qbef ERROR error='Uncaught Error: BOOM!'
line=87 location='http://localhost:8100/' scryo-set/scryo-webapp-dev
{% endhighlight %}

With stacktracejs, I could get:

{% highlight javascript %}
23 Apr 2016 09:54:18.090 196.215.130.135 lwklfalg
ERROR message='Uncaught Error: BOOM!' stack.0='three()@http://localhost:8100/:84:21'
stack.1='two()@http://localhost:8100/:81:9' stack.2='one()@http://localhost:8100/:78:9'
stack.3='{anonymous}()@http://localhost:8100/:76:5' scryo-set/scryo-webapp-dev
{% endhighlight %}

It must be mentioned that Chrome itself does a decent job of showing the stacktraces
when you develop locally. The problem comes when you try to get the stacktrace remotely.
It's exactly for this kind of purpose that services like TrackJS exists.

I had yet to properly log Angular bugs. The same principle for logging errors is true here: Debugging
within Chrome is easy as errors are clearly reported. Remote logging is the difficult part. I simply
called, on initialisation of one of my controller, an undeclared function:

nonExistentFunction();

With my initial remote logging code, I could log the following remotely:

{% highlight javascript %}
23 Apr 2016 07:15:05.766 196.215.130.135 tiazyemy
LOG error=undefined message='nonExistentFunction is not defined'
cause=
<ion-nav-view name="tab-dash" class="view-container tab-content" nav-view="active" nav-view-transition="ios">
scryo-set/scryo-webapp-dev
{% endhighlight %}

With improved StackTraceJS logging, I was able to get:

{% highlight javascript %}
23 Apr 2016 10:31:32.211 196.215.130.135 iepyjgdy ERROR
message='nonExistentFunction is not defined'
stack.0='new DashCtrl()@http://localhost:8100/app/app.js:542:9'
stack.1='e()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:73:156'
stack.2='Object.instantiate()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:73:273'
stack.3='{anonymous}()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:114:228'
stack.4='I.appendViewElement()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:447:4293'
stack.5='Object.O.render()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:446:17421'
stack.6='Object.O.init()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:446:16656'
stack.7='I.render()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:447:3249'
stack.8='I.register()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:447:2980'
stack.9='a()@http://localhost:8100/lib/ionic/js/ionic.bundle.min.js:448:8127' scryo-set/scryo-webapp-dev
{% endhighlight %}
