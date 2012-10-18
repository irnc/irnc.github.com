---
layout: post
title: Overview of Derby's internals
---

# What each Derby module is responsible for?

*   ## derby.js

    Project's main module. Exports object which is prototyped from `racer`
    module, made pluggable under a name `derby` and decorated with one common
    plugin (exported from `./component` module) and a plugin exported from one
    of `./derby.server` or `./derby.browser` modules depending on the execution
    environment. This object is referenced below as *`derby` object*.

    See [Racer modules overview](/2012/10/16/racer-overview.html) for details
    regarding Racer and it's `./plugin` module.

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

    **file** parameter is a path (String) pointing to a file with an
    *application server module*.

    **port** parameter specifies port number on which master HTTP server will
    be listening.

    **options** is an options object passed to the 'up' npm mpdule with
    possible properties as follows:

    * numWorkers {Number} defaults to 1

    This method creates the master HTTP server and starts cluster with a
    specified number of workers using the 'up' npm module.

    Note: master HTTP server starts listening on port even before the up cluster
    is created.

    ### function createApp(appModule)

    * merges in EventEmitter's prototype to app's module exports, thus making
      the *application object* an event emitter/listener;
    * creates *view object* passing in libraries array from *component plugin*
      and *application object* to View constructor exported from ./View.server
      module;
    * exposes derby.settions property to view object via view._derbySettings;
      view also given access to appModule.filename via _appFilename;
    * sets application.{routes,view,ready,render} properties;
    * calls view.render() without parameters on the next tick so that files
      will be cached for the first render (note that view.render()
      implementation differs in server and browser contexts).

*   ## derby.browser.js

    In browser there are only one derby, view, model and page objects and they
    are stored in propected properties (as defined by Crokford) in this module.

    *Derby's browser plugin* exposes *derby object* to be accessible via
    `window.DERBY` property and adds `createApp()` and `init()` methods to it.

    Internals of `createApp(appModule)` in *Derby's browser plugin*:

    * merges in EventEmitter's prototype to app's module exports, thus making
      the *application object* an event emitter/listener;
    * creates *view object* passing in libraries array from *component plugin*
      and *application object* to View constructor; view object is stored into
      derby.view, application.view and to protected property.
    * setups tracks by calling `tracks.setup(app, createPageFn, onRouteFn)`;
    * sets view.history to application.history;
    * defines application.ready(fn) function, where fn will be called upon
      application object with model as a parameter on racer's ready event.

    TODO: find out who calls derby.init()

    derby.init() method's internals:

    * listens for racer's init and ready events;
    * calls racer.init(modelBundle).

    ### Page

    This object's prototype is defined differently for server and browser.

    On browser it is reduced to empty constructor. Resulting Page object has
    render(ns, ctx) method which is a proxy to view.render(model, ns, ctx). So
    essentially Page object bounds model to view object.

*   ## refresh.server.js

    Exports object containing autoRefresh() method.

*   ## View.js

    Module exports View object constructor.

    ### View(libraries, application)

    Stores passed in libraries array into `view._libraries` and *application
    object* into `view._appExports` property.

    ### inline

    Each View object, either in browser or server has `inline` method and
    `_inline` property initialized to empty string by constructor.

    In browser context `inline` is an empty function and `_inline` property is
    never used. See description of the View.server module about implementation
    of the `inline` method used in server context.

    ### view.render(model, ns, ctx, silent)

    Alternatively can be called as view.render(model, Object ctx, silent) in
    that case, ns is assumed to be an empty string.

    Internal workings:

    * assigns passed in model argument to this.model;
    * sets this.dom._preventUpdates to TRUE;
    * stores ns and ctx arguments to the this._lastRender object;
    * TODO: mention what is done here
    * if silent evaluates to TRUE, stop here;
    * removes all attributes from `<html>` element in window.document; sets
      html elements's attributes from rendered root template to document's html
      element;
    * replaces body of a window.document with rendered body element;
    * sets document.title from title view template;
    * calls this._afterRender(ns, ctx).

    ### view.get (name, ns, ctx)

    If *ns* parameter is omitted it is assumed to be an empty string. Context
    object is extended using default context. If ctx parameter is omitted,
    context object will be created entirely from the default context.

    On resulting context object '$fnCtx' property is set to
    `[view._appExports]`.

    Method returns the result of calling `this._find(name, ns)(ctx)`.

    ### view._find (name, ns, macroAttrs)

    When macroAttrs parameter is not set (as when calling from view.get()
    method), this method is just a proxy to `view._findItem(name, ns, '_views')`
    with one addition: `_find()` will throw an 'Can't find view ...' error if
    return value from call to view._findItem will evaluate to FALSE.

    ### view._findItem(name, ns, prop)

    If ns evaluates to FALSE, just lookup for *name* property in `view[prop]`
    object and return it, i.e. return undefined it property does not exists.

    If ns is set, try to find item with name prefixed with it walking upwards,
    e.g. if ns is set to 'x:y:z' and name is 't', lookup will be done for those
    properties in the order specified: 'x:y:z:t', 'x:y:t', 'x:t', 't'.

    ### ctx = view._beforeRender(ns, ctx)

    Create context object if it is not set, store *ns* to `ctx.$ns`

    Emits 'pre:render' event on application object with context object as a
    parameter. If `ns` evaluates to TRUE, also emits 'pre:render:*ns*' event
    with the same context object as a parameter.

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

    ### view.render(res, model, ctx, ns, resStatusCode, isStatic)

    *res* parameter is used to pass in response object, if not set it will be
    mocked by *empty response object*.

    *model* must be an instance of `racer["protected"].Model` otherwise empty
    model object will be used with empty `_commit` and `bundle` methods.

    Rest of the arguments will be assumed to be undefined unless they match the
    criteria:

    * ctx is of type object
    * ns is of type string
    * resStatusCode is of type number
    * isStatic is of type boolean

    After that arguments normalization view.render will call

    ctx = this._beforeRender(ns, ctx); // defined in the View module

    this._init(model, isStatic, function() { // defined in View.server module
      view._render(res, model, ns, ctx, isStatic);
    });

    view._init method calls view._load and when it will be calling back, code
    in _init will partially mock model with empty implementations and call
    view._initValues(model) and callback().

    #### view._init

    This method is called only from `render` method.

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

*   ## eventBinding.js

    Parses x-bind definition and adds events to the DOM. Module is used by
    markup.js to add DOM events from parsed markup.

*   ## markup.js

    Defines parsers for Derby's template markup. Module is used by View.js

# Core application modules

*   server

    The sole responsibility of this module is to call derby.run('lib/server'),
    i.e. running Derby and delegating all work to the *server module*.

*   lib/server

    This module is called the *server module*.

    It creates a HTTP server (without listening on port), initializes an
    express application and includes Connect middleware.

    *Server module* must export an instance of http.Server. Otherwise call to
    `derby.run` will throw an error.

*   lib/app

    TODO

# Environment variables

*   PORT

    Port to which bind HTTP server. If it is not set, and NODE_ENV is set to
    "production", server will be bind to port 80, otherwise on port 3000. See
    function run() inside the `lib/derby.server` module.

# Notes about external dependencies

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
