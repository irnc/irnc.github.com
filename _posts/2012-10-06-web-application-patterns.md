---
layout: post
title: TODO Pattern applied to Web application
---

# Model-View-Controller (MVC) pattern

Assumes:

* a long-lived view and ([nodejitsu-isomorphic][])
* a swappable Controller ([nodejitsu-isomorphic][])

Definition:

* it is a design pattern for user-facing software ([mvc-wikipedia][])
* separates the representation of information from the user's interaction with it ([mvc-wikipedia][])
* user interaction is coming into the View ([nodejitsu-isomorphic][])
* View passes calls to Controller ([nodejitsu-isomorphic][])
* Controller manipulates Model ([nodejitsu-isomorphic][])
* Model fires events on View ([nodejitsu-isomorphic][])

Notable implementations:

* Backbone.js uses slightly modified version of a pattern ([nodejitsu-isomorphic][])

History:

* The model-view-controller pattern was originally formulated in the late 1970s
    by Trygve Reenskaug at Xerox PARC, as part of the Smalltalk system.

My conclusions:

TODO

[mvc-wikipedia]: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[nodejitsu-isomorphic]: http://blog.nodejitsu.com/scaling-isomorphic-javascript-code
