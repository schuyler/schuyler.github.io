---
layout: post
title: "How to install Ubuntu packages non-interactively."
date: 2012-08-30 18:26
comments: true
categories: ubuntu osm2pgsql dpkg
---

God bless the German OpenStreetMap developers, because enough of them like
either Debian or Ubuntu that they ship their source code with Debian packaging.
Building [osm2pgsql](http://wiki.openstreetmap.org/wiki/Osm2pgsql) becomes a
simple matter of doing this:

``` bash
$ svn co http://svn.openstreetmap.org/applications/utils/export/osm2pgsql/
$ cd osm2pgsql
$ debuild
```

If you do this, you wind up with two Debian packages:

``` bash
$ ls ../*.deb
../openstreetmap-postgis-db-setup_0.80.0-13~precise1_all.deb
../osm2pgsql_0.80.0-13~precise1_amd64.deb
```

Which is great, because then you can install
`openstreetmap-postgis-db-setup` using, say, `dpkg -i`, and it'll
automatically create a `gis` user and a `gis` database with PostGIS
extensions for use with `osm2pgsql`. The not-so-great part is that the
package install asks you a bunch of questions about what you want to call
the database and the PostgreSQL user, which is fine, until you're trying to
automate the process of building a Planet.osm database on EC2, and then it
means your script will hang on a silly curses prompt.

<!-- more -->

The solution I found by searching the Internet is to [use the
DEBIAN_FRONTEND environment variable](http://serverfault.com/questions/197495/ubuntu-dpkg-non-interactive-installation):

``` bash
$ DEBIAN_FRONTEND=noninteractive dpkg --install ../openstreetmap-postgis-db-setup_0.80.0-13~precise1_all.deb
```

Which is great if you're root. But try to do it with `sudo` and you're
boned, because `sudo` clears the environment as a security measure. Here's
the fix for that:

``` bash
$ sudo su -c 'DEBIAN_FRONTEND=noninteractive dpkg --install ../*.deb'
```

This uses `sudo` to start a root shell via `su` and then executes `dpkg`
with the right environment variable from there. Unwieldy, but it works.

