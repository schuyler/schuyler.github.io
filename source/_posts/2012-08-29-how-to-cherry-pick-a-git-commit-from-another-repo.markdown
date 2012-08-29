---
layout: post
title: how to cherry-pick a Git commit from another repo.
date: 2012-08-29 15:43
comments: true
categories: git
---

So, as soon as I got this blog started, I discovered that the Github plugin on
the right side wasn't showing any repos. Naturally, I googled for the
[error message](https://github.com/imathis/octopress/issues/636) and found
[a reference](https://github.com/imathis/octopress/issues/636#issuecomment-8113731) to 
[a pull request](https://github.com/imathis/octopress/pull/732) for
[a commit](https://github.com/djohnsonm/octopress/commit/dd8b24f94c6cb86124d7bc3c8cf0ada6c974e214)
that fixed the issue. How to apply David Johnson's fix without possibly
screwing up everything else in my local Octopress repo?

First, add his fork as a remote, fetch it, and make sure that the desired
commit is in there.

```
git remote add djohnsonm https://github.com/djohnsonm/octopress.git
git fetch djohnsonm
git log djohnsonm/master
```

Sure enough, the fix is at the top.

```
commit dd8b24f94c6cb86124d7bc3c8cf0ada6c974e214
Author: David Johnson <djohnson.m@gmail.com>
Date:   Tue Aug 28 21:31:20 2012 -0500

Update .themes/classic/source/javascripts/github.js

Fixed the github repo url to return an valid url and not just .json.
```

So `git cherry-pick` to rescue, then just push the current branch back up to
_my_ origin repo.

```
git cherry-pick dd8b24
```

That's all you have to do as far as Git is concerned.

In terms of fixing the issue in Octopress, note that this doesn't actually
update `source/`, just `.themes/source/`, so copy the file manually, commit,
and push.

```
cp .themes/classic/source/javascripts/github.js source/javascripts/
git commit -m "Deploy the applied github.js fix."
git push
```

Then, since this is Octopress, rebuild the site and deploy.

```
rake generate
rake deploy
```
