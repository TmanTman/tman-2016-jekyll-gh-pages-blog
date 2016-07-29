---
layout: post
title: Ionic and Firebase lifecyle management - onAuth and onResume
---

Managing your initialization code can be tricky business. This blogposts looks at the different
entry points to an Ionic application.

The first entry point is when the Ionic (therefore Angular) application starts up. All the
.run code sections execute, as is part of the Angular lifecycle. This is a good initialization
point where you can call initialization code in your various angular services.

In a .run section for my application, I put the code below to detect the Cordova and Firebase
lifecycle/authentication events. The two events broadcasted on the rootscope is done on
Firebase.onAuth changes, respectively for the case where the application authenticats against Firebase
or where it unauthenticates from Firebase.

{% highlight javascript %}
.run(function($rootScope) {
    if (document.URL.indexOf( 'http://' ) === -1 && document.URL.indexOf( 'https://' ) === -1) {
        //Native app
        document.addEventListener('pause', onPause, false);
        document.addEventListener('resume', onResume, false);
    } else {
        //Web browser app
    }

    $rootScope.$on('auth:authed', loggedInFunction);
    $rootScope.$on('auth:not_authed', loggedOutFunction);

    function onPause () {
        console.log('TEST: onPause');
    }

    function onResume() {
        console.log('TEST: onResume');
    }

    function loggedInFunction() {
        console.log('TEST: loggedInFunction');
    }

    function loggedOutFunction() {
        console.log('TEST: loggedOutFunction');
    }
});
{% endhighlight %}

This code is first tested on a browser and then on a mobile device. The starting point for the application
is where we still need to log in to the application.

On login:
TEST: loggedInFunction
When I explicitly log out:
TEST: loggedOutFunction
When I log in (not in Incognito mode), exit the tab/browser and reopen it at the app's URL:
TEST: loggedInFunction

That's it for the browser! Nice and easy. On an Android device, the cordova onPause and onResume
methods complicate the lifecycle/auth events just a bit more.

Open app after logout:
TEST: onResume
Login:
TEST: loggedInFunction
Logout:
TEST: loggedOutFunction
Close app and open again:
onPause
onResume
Press the Home button without logging in:
onPause
Open the app again:
onResume
Login:
loggedInFunction
Press the Home button while logged in:
onPause
Open the app again:
onResume

This shows that authentication is *not* checked when the app is resumed. So what happens when the
authentication token expires while the app is paused? What happens on resume?
Login, Press Home screen button, Let auth expires. Then onresume:
Auth token immediately revoked and the loggedOutFunction is invoked.

Awesome! That's a wrapup of the lifecyle and auth events of Cordova and Ionic.
