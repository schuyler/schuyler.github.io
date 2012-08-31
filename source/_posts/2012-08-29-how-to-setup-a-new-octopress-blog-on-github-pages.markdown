---
layout: post
title: "How to setup a new Octopress blog on Github Pages."
date: 2012-08-29 18:10
comments: true
categories: github octopress yak
---

The yak shaving started when I wanted a new blog that was easy to update, easy
to maintain, didn't depend on anything fancy, wasn't my personal blog (still
running on blosxom), and wasn't WordPress. Enter [Octopress](http://octopress.org/),
whose virtues I will let speak for themselves.

Here is a rapid-fire Octopress Github OS X tutorial on how to get Octopress
publishing to Github Pages from OS X. I refer you to the
[Octopress documentation](http://octopress.org/docs/)
for actual detailed instructions.

<!-- more -->

### Octopress wants Ruby 1.9.3 and OS X comes with 1.8.7.

Solution: [Something called RVM](http://octopress.org/docs/setup/rvm/). Ruby
Version Manager? Who cares.

```
$ curl -L https://get.rvm.io | bash -s stable --ruby
```

This installs a bunch of crap (i.e. Ruby 1.9) to `~/.rvm/` in your home
directory but whatever. It also has the somewhat unsavory side effect of adding
this to your `.bashrc`:

```
PATH=$PATH:$HOME/.rvm/bin # Add RVM to PATH for scripting
```

Hey! Thanks for letting me know, jerk. Well, source your `.bashrc` to get the
new version of Ruby and move on.

```
$ source ~/.bashrc
```

### Get Octopress and its dependencies.

```
$ git clone git://github.com/imathis/octopress.git my_new_blogue
$ cd my_new_blogue
$ bundle install
```

This creates the local repo that will contain your new static weblog
by [cloning imathis's Octopress source tree](http://octopress.org/docs/setup/) into `my_new_blogue/`,
and then chdirs into it and installs its delightful Ruby prerequisites. Into `~/.rvm` again. I
think.

### Copy the classic Octopress theme.

From inside `my_new_blogue/` (or whatever you called it):

```
$ rake install
```

You want a different theme? Figure it out yourself, kid. Look in
the `.themes/` directory.

### Fix some bugs in Octopress manually.

Hopefully, you are reading this far enough in the future that these
workarounds are no longer necessary. However, if not, then you
will want to
[fix the busted Github plugin](/blog/how-to-cherry-pick-a-git-commit-from-another-repo.html) and
you probably will want to [fix list indentation](https://github.com/imathis/octopress/issues/417). I hope that
imathis gets around to fixing this stuff in HEAD. Moving along...

### Setup to deploy to username.github.com

```
$ rake setup_github_pages
```

This does a bunch of stuff, which is
[documented](http://octopress.org/docs/deploying/github/)
but the part you care about is where it asks you for
the repo URL, tell it `git@github.com:username/username.github.com` while
substituting as necessary.

### Configure your shiny new blog.

[Configure the thing](http://octopress.org/docs/configuring/) by editing the `_config.yml`
file in the top-level directory. I only changed the following settings. You
probably want to leave everything else alone, to start.

+ `url`
+ `title`
+ `subtitle`
+ `author`
+ `description`
+ `github_user`
+ `twitter_user`
+ `googleplus_user`
+ `disqus_short_name`

You'll only want to set a `disqus_short_name` if you want comments. You need to
[sign up](http://disqus.com/) for the Disqus service, of course. I thought it
wasn't a bad idea. Lack of comments was the main thing I came to dislike about
blosxom, pretty much.

Other stuff I monkeyed with but you probably shouldn't unless you know what
you're doing and why:

+ `date_format`
+ `permalink`
+ `paginate`
+ `excerpt_link`
+ `titlecase`
+ `github_repo_count`
+ `github_skip_forks`
+ `twitter_tweet_count`
+ `googleplus_hidden`
+ `facebook_like`

I set `date_format` to `"%F"` because this is a hacker blog and ISO 8601, so there.

I changed the permalink format because why _also_ put the date in the URL.  _Do
not_ bother taking `/blog/` out of the permalink, because that part is
hardcoded into the archive URL, so _le sigh_.

I turned off `titlecase` because I like my titles lowercased, thank you.

I added my G+ ID and FB liking because I am kind of a whore that way. I turned
off Tweet count because I am not _that_ much of a whore.

### Writing new posts.

```
$ rake new_post["This is the title of my new post."]
```

Then you gotta find the filename it prints out under `source/_posts/` somewhere
and go [edit that file with Markdown](http://octopress.org/docs/blogging/) or something.

Running `rake preview` and then visiting
[http://localhost:4000/](http://localhost:4000/) is kind of nice because
then you actually can see what it's all going to look like before you upload to
Github Pages. A+++ WOULD PREVIEW AGAIN.

### Publishing.

```
$ rake generate
$ rake deploy
```

Don't ask me why these are two separate steps. Now you can go look at
http://username.github.com/ or whatever and voilà your new blogue.

BUT WAIT. You're not done yet. You've deployed the _content_ but you still have
to commit your original writings in Markdown format to Github for all
prosperity. This happens in a separate `source` branch (which if you run `git
branch` you will see is already checked out locally, thanks to `rake
setup_github_pages`).

```
$ git add .
$ git commit -m "I made a post." -a
$ git push
```

In other words, your Markdown goes in the `source` branch and the resulting HTML
winds up in the `master` branch, which is what Github is supposed to publish
at your URL. If this doesn't happen -- and it should have already, if you did
everything in the given order -- then
[please note the following](http://octopress.org/docs/deploying/github/):

> Note: With new repositories, Github sets the default branch based on the
> branch you push first, and it looks there for the generated site content. If
> you’re having trouble getting Github to publish your site, go to the admin
> panel for your repository and make sure that the master branch is the default
> branch.

At any rate, Github gets to keep both branches, so that you can always
remake one from the other.

### FOR BONUS POINTS: Custom Domains.

Set your `A` record for `my-blogue.com` to point to `207.97.227.245` (this is
an IP address for `pages.github.com`) and your `CNAME` for `*.my-blogue.com` to
point to `username.github.com`. _Don't_ set your main domain to a `CNAME`
instead of an `A` record, because don't, that's why.

Then do:

```
echo 'myblogue.com' >> source/CNAME
rake generate
rake deploy
```

And wait ten minutes. Seriously, that's it. While you're waiting, don't forget
to:

```
git add source/CNAME
git commit -m "Hooray, my very own domain"
git push
```

### NOW GO BLOGUE AWAY.

And, for God's _sake_, change the `favicon.png` right now.

Remember when I said this would be a rapid-fire tutorial? SHUT UP.

