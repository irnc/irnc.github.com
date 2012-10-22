---
layout: post
title: Building nodejs-0.8.11 as a package on Debian
---

Because of Derby dependency on node.js 0.8.0 and Debian containing only 0.6.19
version pre-build in a repository, the need to build nodejs package from sources
arisen.

Here is a short list:

*   Clone needed Debian package repositories

    * `git clone git://git.debian.org/git/collab-maint/nodejs.git`

    On October 6, Jeremy Lal added commit bc9b8ec9 which states "B-D libv8-dev
    \>= 3.11, fixing for earlier versions is doable but not easy." Because even
    experimental repository does not have pre-build libv8-dev 3.11, we need to
    build libv8 package too.

    * `git clone git://git.debian.org/git/collab-maint/libv8.git`

*   Building libv8 package

    TODO

*   Building nodejs package

    * `cd nodejs`
    * install build dependencies
      * run `dpkg-checkbuilddeps` to see unmet dependencies and install them
      * `apt-get install git-buildpackage pristine-tar gyp`
      * pristine-tar is needed because Debian repository does not have pristine-tar branch and it must be regenerated
    * build the package `git-buildpackage --git-export-dir=../build-area/ -us
      -uc --git-export=remotes/origin/master-experimental`
