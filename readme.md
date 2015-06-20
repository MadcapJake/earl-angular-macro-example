# Earl Grey Angular.js Macro & Example App

In an attempt to learn [Earl Grey](http://breuleux.github.io/earl-grey/) macros, I went off working on a question on EG's [gitter](https://gitter.im/breuleux/earl-grey):

> Is there a way to force a variable name that doesn't append the block level to the end.

> My issue is that I'm interfacing with a library that reads the names of the parameters that are passed into it's callback function.

> It's a common pattern with Angular.
```JavaScript
app.factory("Chats") with ($rootScope) ->
    ;; do something
```
> the `$rootScope` param is read by the framework so that it knows to inject that service.

See, Earl Grey mangles variable names so Angular was throwing errors that it can't find `$rootScope` (or `$routeProvider` with [ngRoute](https://docs.angularjs.org/api/ngRoute)).

I had quite the time trying to remember angular as the last time I used angular was three years ago and it was actually [AngularDart](https://angulardart.org/).  I don't think I even used dependency injection back then (I was quite the beginner).  

With the help of @davej and a [couple](http://jerodsanto.net/2013/08/you-dont-have-to-annotate-your-angularjs-injections-anymore/) [articles](http://taoofcode.net/studying-the-angular-injector-annotate/), I worked out a small solution using EG's `inline-macro`.

## The Example App

The example app was taken from Curren Kelleher's "Introduction to Angular.js in 50 examples" [screencast](https://www.youtube.com/watch?v=TRrL5j3MIvo&feature=youtu.be).  I didn't get a chance to watch the full video but the examples were incredibly helpful and it was pretty straightforward getting it rewritten in EG.

The specific example I used is [here](https://github.com/curran/screencasts/tree/gh-pages/introToAngular/examples/snapshots/snapshot41). In my rewrite, `server.js` is rewritten and the inline client-side script in `index.html` is pulled out into `client.eg` which then gets compiled to js with browserify and [earlify](https://github.com/breuleux/earlify).  `server.eg` isn't compiled because earl can run it perfectly well.

## Run It

First make sure you have iojs and Earl Grey (`--global`) installed. For iojs, [nvm](https://github.com/creationix/nvm) is really nice and easy (linux user here, I don't know about other platforms).

If you want the easiest bash experience, I would also suggest installing browserify and earlify globally.

*If* you went the global route, running this app is as easy as:

```bash
npm install
browserify -t earlify client.eg > client.js
earl run server.eg
```

## The Macro

My macro uses the `macro x: y` format and the first argument is either a bare argument or a list of arguments.  The second argument is the body of a function that the macro creates.  The macro returns a list with all of the  argument names as strings and then the *last* argument is a function with the argument names again and then the function body.

The example above becomes:
```
app.factory("Chats") with
  annotate $rootScope:
    ;; do something
```
And as stated you can replace `$rootScope` with a list of args.  The macro does require using the `with` keyword as it joins the first argument `"Chats"` to the list that it creates.

## The Next Step

You could easily take the macro and move it out into a separate file as a regular macro.  You could expand the macro's patterns to include the name and get something like this:

    app.factory annotate Chats($scope, $param, arg):
      ;; do something

If you're interested, fork this and check EG's docs on [macros](https://breuleux.github.io/earl-grey/doc.html#macros).

## An Alternative Route

On gitter, @breuleux and @davej talked of using a high-level `angular` macro that would search through the nested code but I didn't feel that I had the macro chops to do that. Always better to start small especially when you're learning.

In theory, this would allow you to make a full-fledged DSL for Angular and would be really cool.  One suggestion was a syntax like:
```
require-macro:
   angular-earl

globals:
   angular

app = angular.module("my-app-name")

angular-earl app:
   factory Chats: ->
      require:
         $rootScope as "$root"
         $http

      ;; do something

;; Or use with a class
angular-earl app:
   factory class Chats:
      require:
         $rootScope as "$root"
         $http

      constructor = ->
         ;; do something
```
Definitely would be a great way to utilize Earl Grey's macros, so consider taking the plunge if you want to see something like it!

## Development Tips

* `earl run` gives you nice highlighted error messages that browserify just plugs into your js file.  So I always do a `earl run` before compiling.  If you see `TypeError: Cannot read property 'module' of undefined` it means that your syntax works and it's begun running the app (which won't work cus it isn't browserified and in a browser).
* The next step is to compile and run it to see if Angular is happy. The Angular error messages are pretty neat and I love how they even plug the error and some details of your app right into their website.
* Atom has Earl Grey [syntax highlighting](https://github.com/MadcapJake/language-earl-grey) (written by me!). It's definitely not done and it can be slow, but it's serviceable.
* Also of use is [atom-runner](https://github.com/lsegal/atom-runner).  In my `config.cson`, I have:
```coffeescript
  runner:
    scopes:
      "earl-grey": "env PATH=/path/to/io.js/v2.0.1/bin:$PATH earl run"
```
* This let's me hit `alt-r` and instantly get some results. Obviously not useful for browser projects but I usually create a "sandbox" file and copy over the troublesome code, create some mock data, and use a whole bunch of print statements!  It's really slick.

## Earl Grey Is Awesome!
So there's my brief post-mortem on building a macro for Angular in Earl Grey. I hope you check out Earl Grey!  It's a great little planet in the [JS Verse](https://brendaneich.com/2015/06/from-asm-js-to-webassembly/)!
