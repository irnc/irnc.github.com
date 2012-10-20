---
layout: post
title: [TODO] Model security in Derby framework
---

On April 25, 2012 [an issue][issue #47] about security has been raised: Derby
exposes all model data to a client.

What is needed to fix this issue is a way to authorize model paths.

On July 16 Nate closed [issue #47][] because the first version of access
control has been implemented in Racer, documented in a [accessControl/README][].

rma4ok rumored that "You don't have server-client logic separation. To be
exact, you have some client only logic, but don't have server-only."

## How to store server-only code and data?

* Racer's plugin.useWith object

# Enabling access control for the model

Model in Derby is provided by Racer engine. It is always exposed to clients as
`window.DERBY.model` JavaScript object, so it should only contain data and
allow operations that user is granted access to.

With is done via build-in accessControl plugin. To enable access control
(which is disabled by default) on the store, set `store.accessControl` to
`true`. After that access rules should be defined.

The process of defining access control rules is described in a [README file][accessControl/README].

[issue #47]: https://github.com/codeparty/derby/issues/47
[accessControl/README]: https://github.com/codeparty/racer/blob/master/src/accessControl/README.md
