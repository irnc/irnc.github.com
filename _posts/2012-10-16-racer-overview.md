---
layout: post
title: Overview of Racer's internals
---

# Core concepts

Racer use a few of module patterns for code structuring.

## Plugin

TODO

## Mixin

TODO

# Internal modules

*   ## ./racer

    Project's main module. Exports an extended EventEmitter object.

    After creating aforementioned object, `./plugin` module is merged into it,
    same with object literal consisting of: `version`, `isServer` & `isClient`
    properties, links to `./util` and `./transaction` modules and `uuid`
    function. So exports from `./plugin` module and specified properties can be
    used right from `racer` module's exports (and thus from `derby` module's
    exports).

    ##  ./racer.server

    Server-side part of a main racer module is made as a plugin to it.

    It adds `createStore` method to racer object.

    ### createStore(options)

    Create new Store object; calls store.listen() and emits 'createStore' event
    on the racer object.

*   ## ./plugin

    Allows to register `object` as pluggable under a `name` via call to
    `_makePlugable(name, object)`.

    Later these objects can be instructed to use plugins by calls to
    `use(plugin, options)`. `plugin` can be a `String` in a server environment,
    but must be a `Function` of `(pluggableObject, options)` in browser,
    otherwise `use()` will silently return without doing anything.

    A function representing `plugin` must have a `decorate` property to specify
    which object to decorate, it can be `null` or a `String` used as a `name`
    in call to `_makePlugable()` (e.g. `racer` or `derby`).

*   ## Store

    ### Constructor

    Creates database adapter and stores it into `_db` property. If `options.db`
    is not specified, Memory adapter will be used.

    Adapter is created using createAdapter function exported from `./adapters`
    module.

*   ## ./adaptors

    ### registerAdapter(adapterType, type, adaptorKlass)

    adaptorType can be a 'db', 'clientId' or 'journal', otherwise TypeError
    will be thrown.

    ### createAdapter(adapterType, opts)

    If Adapter object will have a `connect` method, it will be executed here.

*   ## ./session

    This module is implemented as a plugin which mixins into racer's Store.

    See `./session/session.Store` module for documentation about thelifecycle
    of a session.

# Environment variables

*   NODE_ENV

    If set to "production", util.isProduction will be set to TRUE. See
    lib/util/index module for implementation.

# Notes on racer-db-mongo package

It uses `mongoskin` package to acess MongoDB.

Database adapter registered by racer-db-mongo has `connect` method.