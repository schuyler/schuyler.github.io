---
layout: post
title: "How to permanently remove files from a Git repository."
date: 2012-09-06 10:22
comments: true
categories: git
---

Oops. I did a `git commit .` and then the subsequent `git push` took
suspiciously long. Yep, that's right, I've pushed a gigantic binary up to my
remote Git repo by accident. How do I remove the file from the repository
permanently?

<!-- more -->

I found a
[basic solution](http://dound.com/2009/04/git-forever-remove-files-or-folders-from-history/)
given as a shell script, which I just decided to run manually on my local repo.

``` bash
$ git filter-branch -f --index-filter "git rm -rf --cached --ignore-unmatch full/path/to/the/offending.file" HEAD
Rewrite 118bbd4b7b02f01cc221c11aec2705cbxxxxxxxx (59/59)
Ref 'refs/heads/master' was rewritten
$ rm -rf .git/refs/original/ && git reflog expire --all &&  git gc --aggressive --prune
```

You can
[read a little more](http://dalibornasevic.com/posts/2-permanently-remove-files-and-folders-from-a-git-repository)
about how this works, if you like.

This approach worked fine locally, but doing a forced push with `git push -f`
gave me a `[remote rejected]` error, apparently because, as described
in [this StackOverflow post](http://stackoverflow.com/questions/1270514/undoing-a-git-push),
I have the remote repo configured with `receive.denyNonFastForwards`.

Fortunately, this clusterfuck was on a private repository on my own server, so
I could log in and repair the damage manually. If the repo was on Github, I'd have
been [basically out of luck](https://help.github.com/articles/remove-sensitive-data).

So I logged in and ran `su` in order to run commands as the `git` user on my
system (which owns the remote repos).

```
$ sudo su - git
$ cd /path/to/my/repo
```

Then I ran the `git filter-branch` command above on the remote repo, which succeeded,
and then running `git pull` locally also succeeded:

```
 + 119bbd4...eeaxxxxx master -> origin/master  (forced update)
Merge made by recursive.
```

The lesson? Always run `git status` before running `git commit .` or `git
commit -a` -- and use `git commit --amend` judiciously _before_ `git push` if
you screw up! Screwups in the local repo are easier to fix before they get
pushed.
