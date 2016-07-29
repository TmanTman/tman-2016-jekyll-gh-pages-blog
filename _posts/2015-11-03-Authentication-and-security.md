---
layout: post
title: Authentication and Security
---

So the first problem about web security is that no-one really teaches developers about this stuff.
This leads to the large public hackings that took place, like Playstation having all their account
passwords released on PirateBay, since they stored it on their database in plain text. And LinkedIn
and Yahoo and others as well.

I'm in the process of designing a single page app with Angular/Ionic. Other developer will be
my clients, as they will build it into their apps. They will either include
the source code in their projects, or will access by building a webview into their applications and requesting it via
a URL. Notice that I don't care about the current username and password of the end user, I have to
authenticate the current developer that is using my API.

And since I make use of other third-party services, I need to be able to allocate the use of my third-party
services to the current app consuming my services. All without the simple figured-out authentication
method using username-and-password by the end user.

But it couldn't be that hard right? Facebook, GoogleMaps,
Firebase and all those people do exactly this. They provide you with access to their API's.
But alas, there was no single-source-of-truth document that I could read to understand it.
I've had to really grind to figure out exactly how API's and authentication
can be done in a safe way, so that I don't compromise on the security of my clients.

And so I had to learn a lot about web security this week. All the concepts I used to schemed over, I now had to
read up in detail. This includes hashing passwords, doing user management with
PassportJS/DryWall/manually/StormPath/Oauth, understanding tokens vs cookies,
CORS, CFRS and XXS.

And no single guide covers all these, so I reread different sources to try and get to grasp the most of it.
Here I list the best sources that I found:

Youtube - Everything you ever wanted to know about Authentication
Youtube - Tips on designing API's
ScotchIO - Tokens vs cookies
CodeTutsPlus - Tokens vs cookies (only the theory, avoid the example)
