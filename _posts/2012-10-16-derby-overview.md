---
layout: post
title: Overview of Derby's internals
---

## What each Derby module is responsible for?

Update: this post started as a notes on Derby internals, but because of a need
of such documentation was articulated in a [Derby's issue #92][issue-92], these
notes will be added directly to the source code as a docco comments, see
[Derby's pull request #159][pull-159].

So all notes about code will migrate from here to the source code in the near
future.

[issue-92]: https://github.com/codeparty/derby/issues/92
[pull-159]: https://github.com/codeparty/derby/pull/159

*   ## component.js

    Exports *component plugin* which is designed to decorate `derby` object.
    This plugin adds `createLibrary(config, options)` method and `_libraries`
    property to `derby` object.

    `_libraries` is initialized as an empty Array, with `map` property as an
    empty object.

    TODO: describe what `createLibrary()` function does.

*   ## derby.server.js

    Both `./derby.server` and `./derby.browser` modules define *derby plugin*
    designed to decorate `derby` pluggable object with different implementations
    for server and browser environments respectively.

    In server context plugin adds run(), createApp() and createStatic()
    methods to derby object.

    ### function run (file, port, options)

    **file** string retresenting a path to a file with a *server module*.

    **port** number on which master HTTP server will be listening.

    **options** object passed to the UpServer constructor. For available
    options see [up's README][up-github]. If parameter is not set, Derby
    defaults to `{ numWorkers: 1 }`.

    This method creates the master HTTP server and uses the [up][up-github]
    package to start a cluster of a specified number of workers which will
    actually handle requests.

    Each worker is a separate node.js process which requests *server module*
    and binds it to a randomly assigned port. This port will be send to the up
    service when server starts listening on it. For details see
    `up/lib/worker.js` and http://nodejs.org/api/net.html#net_server_address.

    Note: although `listen` method is called on master HTTP server before the
    up workers is created no requests will be unprocessed because `listen` is
    asynchronous.

    Open question: this method requires *server module* just for check of it's
    exports and does not use it after it. Is this require intended and serves
    some purpose other then exports check or not?

    See https://github.com/codeparty/derby/issues/157

    ### function createApp(appModule)

    * merges in EventEmitter's prototype to app's module exports, thus making
      the *application object* an event emitter/listener;
    * creates *view object* passing in libraries array from *component plugin*
      and *application object* to View constructor exported from ./View.server
      module;
    * exposes derby.settions property to view object via view._derbySettings;
      view also given access to appModule.filename via `_appFilename`;
    * sets application.{routes,view,ready,render} properties;
    * calls view.render() without parameters on the next tick so that files
      will be cached for the first render (note that view.render()
      implementation differs in server and browser contexts).

*   ## refresh.js

    ### autoRefresh()

    This method is used only by `derby.browser` module.

    It listens for specific event on model and socket and reloads page or
    partially refreshes HTML or CSS.

*   ## refresh.server.js

    Exports object containing autoRefresh() method.

*   ## View.server.js

    Exports View module constuctor but changes it's prototype by adding isServer
    property set to TRUE and overriding some methods, namely inline and render,
    while adding a bunch of utility methods.

    ### inline

    The `inline(fn)` method casts passed in function to string, wraps it into an
    *immediate invocation* pattern and appends it to the `_inline` proterty,
    meaning that it contains pure JavaScript code, without `<script>` tags.

    In a production environment, string representation of a script to inline is
    uglified by uglify-js required from racer's node_modules directory.

    #### view._load

    This method is called only from `_init` method.

    In a production environment, this method will set view._watch to FALSE and
    override itself with a function which just calling callback. While in
    non-production environtent only `_watch` propery will be set to TRUE.

    After this actual loading will be performed.

    #### view._initValues(model)

    Will set view.model to model, and reset view._idCount to 0. After this, all
    libraries are iterated and `library.view._initValues(model)` is called on
    each.

    #### `view._renderScripts(...)`

    Called only from `view._render` method.

*   ## Dom.js

    Exports Dom constructor. Used only by `derby.browser` module, thus it's API
    is available only in browser context.

    Implementations of an `addListener` and `removeListener` functions are
    defined upon module evaluation, based on browser support of DOM API.
    For browsers supporting only `document.attachEvent`, `addListener` will
    wrap attached callback into additional function in order to normalize event
    object by correctly setting the `target` property so the callback can work
    with the event object in a browser-agnostic way.

    ### Dom(model)

    Creates two distinctive EventDispatcher objects and stores them into
    `_events` and `_captureEvents` properties.

    Adds four methods with privileged access: trigger, captureTrigger,
    addListener and removeListener.

    Stores empty arrays to `_componentListeners` and `_pendingUpdates`
    properties.

*   ## EventDispatcher.js

    Exports EventDispatcher constructor.

    ### EventDispatcher(options)

    Create an object from the prototype which has `clear`, `bind` and `trigger`
    methods.

    Callbacks which will be called on each invocation of `bind` and `trigger`
    can be specified via `options.onBind` and `options.onTrigger`.

    ### clear()

    Sets `names` property to an empty object.

    ### bind(name, listener, arg0)

    Calls `onBind` callback passing in all arguments intact. And stores
    `listener` to `names` property.

    Thus `names` is a data structure of the following format:

        { "click": {
            "listener-as-a-string": Function
          , "another-listener-as-a-string": Function
          }
        , "focus": {
            ...
          }
        }

    Where *listener as a string* is a JSON serialization of a `listener`
    function created using `JSON.stringify(listener)`.

    Open question: why JSON-serialized listener is used as a property name?

    Returns object with listeners, i.e. `this.names[name]`.

    ### trigger(name, value, arg0, arg1, arg2, arg3, arg4, arg5)

    TODO

# Core application modules

*   server

    The sole responsibility of this module is to call derby.run('lib/server'),
    i.e. running Derby and delegating all work to the *server module*.

*   lib/server

    This module is called the *server module*.

    It creates a HTTP server (without listening on port), initializes an
    express application and includes Connect middleware.

    *Server module* must export an instance of http.Server. Otherwise call to
    `derby.run` will throw an error. Such structure of code is required to use
    [up][up-github] package to service live code reloads, see it's README for
    details.

*   lib/app

    TODO

# Environment variables

*   PORT

    Port to which bind HTTP server. If it is not set, and NODE_ENV is set to
    "production", server will be bind to port 80, otherwise on port 3000. See
    function run() inside the `lib/derby.server` module.

# Notes about external dependencies

## up

[Up][up-github] module is described as *Zero downtime reloads for Node HTTP(S)
servers*.

[up-github]: https://github.com/learnboost/up

## browserify

Node.js modules are adapted to browser environment using [browserify][].

[browserify]: https://github.com/substack/node-browserify

## racer

Derby is dased on [racer][] package and it's concepts of plugins.

[racer]: https://github.com/codeparty/racer

# tracks

Routes are made possible to work on server and browser via [tracks][] project,
build by Nate Smith. It is described as a *Shared browser and server routes
built on Express*.

Routes calls Page.render().

[tracks]: https://github.com/codeparty/tracks

# External sources

See [David Rees overview][] about Derby's concepts.

## Model

Is a synchronized store using Redis. Can be optionally backed by MongoDB.

[David Rees overview]: http://www.slideshare.net/studgeek/an-overview-of-derbyjs-and-meteorjs-for-the-nova-nodejs-meetup
