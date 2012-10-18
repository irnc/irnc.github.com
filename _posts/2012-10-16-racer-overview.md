---
layout: post
title: Overview of Racer's internals
---

# Internal modules

*   racer.js

    Project's main module. Exports an extended EventEmitter object.

    After creating aforementioned object, `./plugin` module is merged into it,
    same with object literal consisting of: `version`, `isServer` & `isClient`
    properties, links to `./util` and `./transaction` modules and `uuid`
    function. So exports from `./plugin` module and specified properties can be
    used right from `racer` module's exports (and thus from `derby` module's
    exports).

*   plugin.js

    Allows to register `object` as pluggable under a `name` via call to
    `_makePlugable(name, object)`.

    Later these objects can be instructed to use plugins by calls to
    `use(plugin, options)`. `plugin` can be a `String` in a server environment,
    but must be a `Function` of `(pluggableObject, options)` in browser,
    otherwise `use()` will silently return without doing anything.

    A function representing `plugin` must have a `decorate` property to specify
    which object to decorate, it can be `null` or a `String` used as a `name`
    in call to `_makePlugable()` (e.g. `racer` or `derby`).

# Environment variables

*   NODE_ENV

    If set to "production", util.isProduction will be set to TRUE. See
    lib/util/index module for implementation.