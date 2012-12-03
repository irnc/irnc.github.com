---
layout: post
title: FAQ about how it can be done in Derby
---

`tracks` package which handles route for Derby does not support multiple
callbacks for a route as of now, it would be usefull for subscribing for
data needed in a sidebar for example.

TODO: add multiple-callback support to `tracks`

For now we need to define all routes which will use one callback to
subscribe to user's workspaces to be shown in a sidebar.