---
layout: post
title: Basics for Remote Logging
---

I've just started engaging with users. Looking for a remote logging service with
a decent free usage tier and powerful capabilities, I decided on Logentries.

The first goal was to determine the errors that users run into. In an
Angular application, errors occur in two contexts:

1. In the global javascript window context. The error is then handled by the
window.onerror handler.
2. In the Angular context. Angular, by default, logs the error to the console
and then swallows the error.

The first step is to register a Logentries account and initialize your application
with the token provided:

{% highlight javascript %}
<script>
LE.init({
    token: 'YOUR-TOKEN'
});
</script>
{% endhighlight %}

To log all exceptions that the javascript window (window.onerror) handles, the
initialisation can be modified:

{% highlight javascript %}
<script>
LE.init({
    token: 'YOUR-TOKEN',
    catchall: true
});
</script>
{% endhighlight %}

To test this, I simply call a non-existent function *before* I've bootstrapped
Angular and also outside of the Angular scope. In fact, I'll do it just after my
Logentries initialisation:

{% highlight javascript %}
<script>
LE.init({
    token: 'YOUR-TOKEN',
    catchall: true
});
nonExistentFunction();
</script>
{% endhighlight %}

This then logs the error to Logentries:

{% highlight javascript %}
23 Apr 2016 06:40:17.622 196.215.130.135 w3rrzaq0
ERROR error='Uncaught ReferenceError: nonExistentFunction is not defined'
line=58 location='http://localhost:8100/' scryo-set/scryo-webapp-dev
{% endhighlight %}

The next objective is to log errors from the Angular context. To do this,
I decorate $exceptionHandler in the Angular config lifecycle:

{% highlight javascript %}
(function() {
    angular
        .module('app.core')
        .config(exceptionConfig);

    exceptionConfig.$inject = ['$provide'];

    function exceptionConfig($provide) {
        $provide.decorator('$exceptionHandler', extendExceptionHandler);
    }

    extendExceptionHandler.$inject = ['$delegate', '$injector'];

    function extendExceptionHandler($delegate, $injector) {
        return function(exception, cause) {

            //Original $exceptionHandler functionality is maintained
            $delegate(exception, cause);

            //Log the error to an external service
            LE.log({
                error: exception.error,
                message: exception.message,
                cause: cause
            });
        };
    }
})();
{% endhighlight %}

To test this, I execute an error in Angular: In any controller or service, simply
call a nonexistent function on initialisation. For example:

{% highlight javascript %}
(function() {
    'use strict';

    angular
        .module('app')
        .controller('DashCtrl', DashCtrl);

    DashCtrl.$inject = ['$scope'];

    function DashCtrl($scope) {
        console.log('Calling non-existent function');
        nonExistentFunction();
    }

})();
{% endhighlight %}

This then logs the following to Logentries:

{% highlight javascript %}
23 Apr 2016 07:15:05.766 196.215.130.135 tiazyemy
LOG error=undefined message='nonExistentFunction is not defined'
cause=
<ion-nav-view name="tab-dash" class="view-container tab-content" nav-view="active" nav-view-transition="ios">
scryo-set/scryo-webapp-dev
{% endhighlight %}

With this information, its possible to at least at a basic level track what exceptions occur
in your application when users are using it. A next consideration is to debounce the exception logging
so that one error that is logged 100 times in a second does not cause 100 posts to Logentries. Better stack
traces can also go a long way in better diagnosis of bugs and fixing them quicker.
