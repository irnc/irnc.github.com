---
layout: post
title: How to create a Derby component library
---

## Rationale

Creating in-browser applications requires a large amount of client-side code
and this code must be structured in order to be maintainable.

Derby provides a common way to attach handlers to user events by declaratively
specifying and event handler name through the `x-bind` attribute, e.g.:

    <a x-bind="click: registerClick">Link</a>

Event handling code will lookup context object for 'registerClick' method and
will call it on successful lookup. Otherwise it will throw an Error 'Bound
function not found: registerClick'.

## Actual how to

TODO

## How Derby parses x-bind value?

See splitEvents() function in eventBindings.js

    value = eventBinding[, value]
    eventBinding = eventName[/delay]: callbacks
    callbacks = callback[ callbacks]