---
layout: post
title: Overview of a Derby's tracks package
---

# `tracks` npm package

`tracks` package has two *main* module, one for server and one for browser,
`index` and `browser` respectively.

Both main modules exports `router` module, which exports *router* simple object
(i.e. without a constructor or a prototype) adding `setup` method to it.

Commons in `setup` method in both modules:

* adds `get`, `post`, `put` and `del` *verb methods* to *application object*;

## `router` module

Exports object with the following properties:

* `render` method
* `_mapRoute` method
* `settings` object
* `set` & `get` methods to work on `settions`
* `_isTransitional` method

## `index` module

This is the main module for the package in a server-side environemnt.

### `setup(app, createPage, onRoute)`

Adds get, post, put and del proxy methods to application object; all these
methods are placing received information into internal `routes` array.

Adds `router(options)` method to application object. And return `routes` array.

## `browser` module

This is the main module for the package in a client-side environemnt. This is
specified using `browserify.main` property from `package.json` file.

### `setup(app, createPage, onRoute)`

*   create a *page object* using `createPage` function;
*   create new History object passing in a *page object*;
*   expose that History object to application and page objects via `history`
    and `_history` properties respectively;
*   provide new implementation for `redirect` method of a page object;
*   expose routes to page object via `_routes` property;

Note: this method calls `Array.prototype.forEach` which is not implemented in
all browsers.

### *verb methods*

Note: for each verb method, module maintains a separate queue to which it
pushes routes as they are defined.

Each route in a queue is a Route object created from `express/lib/router/route`
module.

TODO: describe how *[transitional routes][]* are handled.

[transitional routes]: http://derbyjs.com/#transitional_routes

## `History` module

Exports History constructor. If browser is pushState capable, appropriate
listeners will be added to events by `addListeners(history, page)`.

Note: currently tracks does not support browsers which does not support
[pushState][].

[pushState]: https://developer.mozilla.org/en-US/docs/DOM/Manipulating_the_browser_history#Adding_and_modifying_history_entries

### `addListeners(history, page)`

Adds listeners for 'click' and 'submit' events on window.document, handlers will
call `history.push(url, true, null, event)` if all conditions will be met. Here
`true` is for render and null signals that empty state object must be used.

Call to `history.push` will result in calling to `router.render` as a standalone
function with `history.page`, `options` and `event` as a parameters.

TODO: uses `element.addEventListener` which will not be available in IE.

### `window.history.state` object

tracks' History module stores the following properties into the state object:

* `$render` boolean whenever to render route when `popstate` event occurs
* `$method` string representing any verb method, e.g. 'get' or 'put'