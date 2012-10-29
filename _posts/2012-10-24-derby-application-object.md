---
layout: post
title: Derby's application object
---

# Events

The following events are emitted on application object, so you can listen for
them using `app.on('event-name', function callback () {})`:

*   `render`
*   `render:*namespace*`
*   `pre:render`
*   `pre:render:*namespace*`

    Those four events are emitted on an application object by code from
    `view._beforeRender` and `view._afterRender` methods (both defined in the
    `View` module).
