---
layout: post
title: Dependencies of a Derby framework
---

# Engine

Node.js >= 0.8.0 is needed to use Derby 0.3.13 because it is a requirement of
a concat-stream@0.0.8 package which is required in affect from the following
dependency chain:

    derby <- racer <- browserify <- http-browserify <- concat-stream

See [issue #1](https://github.com/maxogden/node-concat-stream/issues/1) where
it is possible to propose pull request to support Node.js 0.6.

# Package dependencies

See [overview post about Derby](/2012/10/16/derby-overview.html) for details
about package dependencies.
