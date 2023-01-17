---
title: "Hugo on Github Pages"
date: 2023-01-16T19:51:56-08:00
---

The earlier iteration of [schuyler.info](https://schuyler.info/) was hosted in
Octopress, which was written in Ruby. One day I tried to write a new post after
leaving the blog untouched for a long time, and long story short, I wound up in
some kind of Ruby module dependency hell. This was like ten years ago, and it
was too much. I shelved the site and never got back to it.

These days it seems like the hotness in static website generation is Jekyll,
but, like, it's Ruby. So no.

One alternative seems to be [Hugo](https://gohugo.io/), which is written in Go
and is one language I trust (sort of) to get dependencies right. I installed it
on MacOS with Homebrew using `brew install hugo`. So far so good.

Another wrinkle was that the Octopress site was already in the `master` branch of 
the [site repo](https://github.com/schuyler/schuyler.github.io). I decided to
start over with a `main` branch using `git switch --orphan` as described in 
this [Stack Overflow post](https://stackoverflow.com/questions/34100048/create-empty-branch-on-github)
followed by `git commit --allow-empty`.

# Install Hugo

The [Hugo set up instructions](https://gohugo.io/getting-started/quick-start/)
seem comprehensive. Need the `--force` flag on the directory because it already
exists.

```sh
hugo new site schuyler.github.io --force
```

# Add a theme

Okay, cool. Gotta have a theme, themes are cool. I choose
[BeautifulHugo](https://themes.gohugo.io/themes/beautifulhugo/).
Apparently you can include the theme as a Git module but no way do I want to do
that, because the source repo might go away. I don't want updates. I want my
blog to stay the same forever, nice and stable, easy to post to any old time.

It's surprisingly challenging to [checkout a bare Git repo with no
history](https://stackoverflow.com/a/71971526) if
the provider doesn't supply a ZIP file. Fortunately I can Google for Stack
Overflow results as good as anyone.

```sh
git clone --depth=1 \
    https://github.com/halogenica/beautifulhugo.git themes/beautifulhugo \
    && rm -rf themes/beautifulhugo/.git

git add themes/beautifulhugo
git commit -m "Add a static version of the BeautifulHugo theme"
```

# Customize the theme

Now BeautifulHugo suggests starting with the example site, so that's what I did.

```sh
cp -r themes/beautifulhugo/exampleSite/* .
hugo serve -D
```

The `-D` flag tells hugo to render draft content.

Then I went through `config.toml`, changing the title, subtitle, and social
media links. I also commented out most of the `menu.main` stuff.

I added an avatar image under `static/underoos.jpg` and then set `Params.logo`
to `underoos.jpg`. I changed the `dateFormat` and uncommented `hideAuthor` to
set it to true. I disabled `socialShare` because ugh.

Finally I renamed `content/_index.md` to `__index.md` to remove the front
matter but not delete the file because otherwise I will forget that it exists.

# Write a first post

Ok. New post with `hugo new posts/slug.md`. Great, it prints out the file name,
now I have to cut n paste it into an editor, bleh. But, it does work. Now to
add this post as well.

# Add .gitignore

I added a `.gitignore` and then checked it all in:

```
*.swp
*.lock
```

# Set up Github Pages build

So, push `main` to `origin`, but now I need to add the Github Action that will
[actually render the site](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

```
mkdir -p .github/workflows
cat > .github/workflows/gh-pages.yml <<End
name: github pages

on:
  push:
    branches:
      - main  # Set a branch that will trigger a deployment
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

Don't forget to update `baseurl` in `config.toml` like the Hugo doc says.

Also, set the site CNAME, commit and push it all up.

```
echo schuyler.info > static/CNAME
git add .github/workflows/gh-pages.yml static/CNAME config.toml
git commit -m "Add GH action and Pages CNAME"
```

# Configuring a custom domain

Following this [custom domain
configuration](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#about-custom-domain-configuration)
doc, I need to go into the DNS for `schuyler.info` and  make sure to set the
correct IPs for the `A` record for my domain, and the `CNAME` record to point
to `schuyler.github.io` (and _not_ `schuyler.github.com` which is what it _was_
set to, which tells you just how long ago I messed with this last).

Finally, go into _Settings → Pages_ in Github, set the publication branch to
`gh-pages`, set the custom domain, wait for the DNS check to succeed
(eventually) and finally click _Enforce HTTPS_ and voilá.

That was definitely not too much work. What is this, 2023 already?
