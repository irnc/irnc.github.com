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

*   ##  ./racer.server

    Server-side part of a main racer module is made as a plugin to it.

    It adds `createStore` method to racer object.

    ### createStore(options)

    Create new Store object; calls store.listen() and emits 'createStore' event
    on the racer object.

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