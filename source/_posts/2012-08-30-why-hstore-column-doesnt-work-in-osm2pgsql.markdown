---
layout: post
title: "Why --hstore-column doesn't work in osm2pgsql."
date: 2012-08-30 18:00
comments: true
categories: OSM osm2pgsql
---

Today I tried using the `--hstore-column` option in
[osm2pgsql](http://wiki.openstreetmap.org/wiki/Osm2pgsql) to import all the
`name:*` tags, because I'm building a gazetteer and I want all of the
various localized names of all the places in OpenStreetMap. The option is
supposed to create an `hstore` column called `name:` in the `planet_osm_*`
tables, with key-value pairs where the key is whatever comes after that, which
is usually the ISO code for the language of the name.

However, when I ran `osm2pgsql` with `--hstore-column "name:*"`, the column
was created but never populated. What the hell?

<!-- more -->

I guess `--hstore-column` isn't widely used, because some adventurous soul
added some code in `output-pgsql.c` which very helpfully strips out tags
from OSM objects that aren't listed in the `.style` file. The giveaway was
this bit of commentary in `pgsql_filter_tags()`:

{% codeblock lang:c %}
/* We used to only go far enough to determine if it's a polygon or not,
but now we go through and filter stuff we don't need */
{% endcodeblock %}

I guess whoever made the change accidentally forgot to check
`Options->hstore_columns`. Understandable. Osm2pgsql's hstore support is
kind of an outlier of a feature. Well, not to worry. I
[fixed it](https://github.com/schuyler/osm2pgsql/commit/c19b6bb52338cddd9dcb9c8d0ecfda85be2b8eda)
in my Github mirror, and
[filed a bug](https://trac.openstreetmap.org/ticket/4547) with a patch in OSM's
Trac. As a bonus, `--hstore-column` once again respects `--hstore-match-only`.

Apparently, I have an OSM SVN account, but it's been so long since I
committed anything that I forget my password, so now I'm waiting for an
admin to reset it. Otherwise, I'd commit the patch myself. Oops!

__UPDATE__ (2012-09-02): TomH was kind enough to reset my password, so
I [committed my patch](https://trac.openstreetmap.org/changeset/28671/applications/utils/export/osm2pgsql) but apmon discovered that a logic bug
caused segfaults when no `--hstore-column` flag was set, so he
[fixed that](https://trac.openstreetmap.org/changeset/28683/applications/utils/export/osm2pgsql). Both changes are now in [Subversion](https://trac.openstreetmap.org/browser/applications/utils/export/osm2pgsql) and in
[my github repo](http://github.com/schuyler/osm2pgsql).
