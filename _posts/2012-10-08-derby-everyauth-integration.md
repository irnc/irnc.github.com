---
layout: post
title: TODO Using everyauth in Derby applications
---

# Current status of authentication in Derby

On August 24, 2012 Nate Smith told "Auth is basically done and integrated with
Everyauth. We just don't have an example yet. I'm kinda hosed for the next
couple of weeks, but coming soon." (see [auth-state-aug-2012][post] at list).

Authentication depends on three different parts of Derby: as by [cjb-july-2012][]

* the new session system (middleware);

    Issues:

    *  the ability to reference the session object in templates

* access control (was introduced in 0.3.12);
* and partly, the new query system.

# Examples

On September 15, 2012 Tyler Renell (@lefnire) presented
[his example from Habit RPG](https://github.com/lefnire/habitrpg/blob/master/src/server/auth.coffee),
and later, at September 21, Laszlo Bacsi (@LacKac) shared
[his approach](https://gist.github.com/3758734 "Authentication with everyauth in Derby by @lackac").



[Carl-Johan Blomqvist][]'s work and [post](https://groups.google.com/forum/#!msg/derbyjs/6CuViVg4p7Q/dKxfuIQQwJAJ "New blog about about authentication and a preview example app")
about everyauth usage with Derby.

[Tyler Renelle][] working on Derby-everyauth integration as a part of his
HabbitRPG project. Currently only Fb authentication is working, Twitter, GitHub
and password authentications [coming next](https://groups.google.com/forum/?fromgroups=#!topic/derbyjs/JuUqUNd9Rls "Problems with password authentication with everyauth").

[cjb-july-2012]: http://cjblomqvist.com/blog/derby-authentication-part-1/23
[Tyler Renelle]: https://github.com/lefnire
[auth-state-aug-2012]: https://groups.google.com/forum/#!topic/derbyjs/7VWTclIBEw8