---
layout: post
title: Deploy Firebase Queue On Heroku
---

If you have work that needs to be done on the server - push it to a queue, and let the server process it from the queue. Firebase has a dedicated open source project to do this all for you - [described fully on the firebase-queue repo](https://github.com/firebase/firebase-queue).

This guide will demonstrate how a simple firebase-queue can be deployed both locally and to a Heroku server.

First, create your own Firebase project on the [shiny new website](https://firebase.google.com/). We'll write a server that will process tasks written to the realtime database.

Lets deploy a node server locally. You'll need to be familiar with npm and bash. Create the directory, initialise the node project with npm:

{% highlight javascript %}
$mkdir test-firebase-queue
$cd test-firebase-queue
$npm init
{% endhighlight %}

Accept all the default settings on the npm init command. Install firebase and firebase-queue libraries:

{% highlight javascript %}
$npm install firebase firebase-queue --save
{% endhighlight %}

Lets create the server code. Create a file in the test-firebase-queue directory called server.js. Copying directly from the github sample, write this sample code into the file:

{% highlight javascript %}
var Queue = require('firebase-queue');
var firebase = require('firebase');

firebase.initializeApp({
  serviceAccount: 'path/to/serviceAccountCredentials.json',
  databaseURL: '<your-database-url>'
});

var ref = firebase.database().ref('queue');
var queue = new Queue(ref, function(data, progress, resolve, reject) {
  // Read and process task data
  console.log(data);

  // Do some work
  progress(50);

  // Finish the task asynchronously
  setTimeout(function() {
    resolve();
  }, 1000);
});
{% endhighlight %}

Some foresight will now be useful. To deploy to Heroku, the initialisation parameters for the firebase.initializeApp cannot be read from a file, Heroku does not support reading from static files on the server. Instead, the values should be read from environmental variables, as is possible through this methods as given on the [Firebase documentation](https://firebase.google.com/docs/server/setup):

{% highlight javascript %}
var firebase = require("firebase");
firebase.initializeApp({
  serviceAccount: {
    projectId: "projectId",
    clientEmail: "foo@projectId.iam.gserviceaccount.com",
    privateKey: "-----BEGIN PRIVATE KEY-----\nkey\n-----END PRIVATE KEY-----\n"
  },
  databaseURL: "https://databaseName.firebaseio.com"
});
{% endhighlight %}

The neccessary details for the abovementioned method can be acquired by going to your Firebase console for your project, at the top of the left sidebar clicking the gear next to the projectId and selecting "Permissions".

The environmental variables for *local* development is set by including a file with the name .env in the root of our test-firebase-queue directory, and using key=value values. I spent hours trying to get this working, even asking help on [stackoverflow](http://stackoverflow.com/questions/38343148/setting-long-environmental-variables-in-nodejs).

What did work? Using [foreman](https://github.com/strongloop/node-foreman). Install it on your system:

{% highlight javascript %}
$npm install -g foreman
{% endhighlight %}

Foreman allows the environment variables to be set in a JSON syntax. The .env file can now contain the following:

{% highlight javascript %}
{
    "firebase": {
        "projectid": "projectId",
        "clientemail": "username@projectId.iam.gserviceaccount.com",
        "privatekey": "-----BEGIN PRIVATE KEY-----\nSoMeKey...EndOfSomeKey-----END PRIVATE KEY-----\n",
        "databasename": "https://projectId.firebaseio.com/"
    }
}
{% endhighlight %}

The server code can now be adapted to read the environmental variables:

{% highlight javascript %}
var Queue = require('firebase-queue');
var firebase = require('firebase');

firebase.initializeApp({
  serviceAccount: {
      "projectid": process.env.FIREBASE_PROJECTID,
      "clientemail": process.env.FIREBASE_CLIENTEMAIL,
      "privatekey": process.env.FIREBASE_PRIVATEKEY
  },
  databaseURL: process.env.FIREBASE_DATABASENAME
});

var ref = firebase.database().ref('queue');
var queue = new Queue(ref, function(data, progress, resolve, reject) {
  // Read and process task data
  console.log(data);

  // Do some work
  progress(50);

  // Finish the task asynchronously
  setTimeout(function() {
    resolve();
  }, 1000);
});
{% endhighlight %}

You can now successfully run your server locally running in your terminal:

{% highlight javascript %}
$nf server.js
{% endhighlight %}

Test it by adding tasks to your firebase as given in the Firebase-queue docs. One method:

{% highlight javascript %}
$curl -X POST -d '{"foo": "bar"}' https://databaseName.firebaseio.com/queue/tasks.json
{% endhighlight %}

Now, on to deploying this to Heroku. Heroku requires a PROCFILE at the root of the application which tells it how to launch the app. But our server is a bit *special*. We'll tell Heroku our server won't receive HTTP requests, it will only work in the background. Our server should be run on a *worker dyno*, not on a *web dyno*. Therefore our PROCFILE will have the following:

{% highlight javascript %}
worker: node server.js
{% endhighlight %}

 Lets put this server on Heroku. To do this, you'll need to be familiar with git. Initialise the repo with git. Then, in addition to library files, do **not** include the .env file in version control - your credentials should never be checked in on version control, to keep it safe. Therefore add the .env file to the .gitignore file. Lastly, add your heroku remote and push your code to it:

{% highlight javascript %}
$git init
$echo node_modules > .gitignore
$echo .env >> .gitignore
$git add .
$git commit -m 'Initial commit'
$heroku git:remote -a your-heroku-repo
$git push heroku master
{% endhighlight %}

Now your Heroku server will start, but it will fail after 60 seconds. Heroku, by default, launches a single *web dyno* even when you **specifically** told it to only run a worker dyno. The web dyno has to bind to the port withing 60 seconds or else the server is shut down. You can see this by checking your heroku logs after 60 seconds:

{% highlight javascript %}
$heroku logs --tail
{% endhighlight %}

To remove the web dyno and restart your server, run the following command (the dyno setting will persist in your directory, I believe):

{% highlight javascript %}
$heroku ps:scale web=0 worker=1
$heroku ps:restart
{% endhighlight %}

Alas, still the server fails, but we only have one more thing to do! The environmental variables has not been set yet for the Heroku environment! To do this, go to your Heroku console, select your app, and proceed to settings:


![Screenshot]({{ site.url }}/assets/heroku.png)


Enter values where Heroku indicates "Config Vars". For the key, as per our environmental variables as used for the local server, we'll have to use FIREBASE_PROJECTID, etc. The values that you used for FIREBASE_PROJECTID, FIREBASE_CLIENTEMAIL and FIREBASE_DATABASENAME can simply be copied. But the FIREBASE_PRIVATEKEY requires some extra work. Copy the entire string into the value field provided. Then, go through the entire string: Everywhere the '\n' sequence appears, erase the two characters and literally insert a line break by pressing the "Enter" key. Save the edits when you're done.

Thats it! Now you can restart your server on Heroku again (again with heroku ps:restart) and it should work :)
