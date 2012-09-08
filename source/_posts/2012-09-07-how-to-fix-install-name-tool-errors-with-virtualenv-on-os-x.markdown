---
layout: post
title: "How to fix install_name_tool errors with virtualenv on OS X."
date: 2012-09-07 17:10
comments: true
categories: python virtualenv osx
---

[This Github ticket comment](https://github.com/pypa/virtualenv/issues/7#issuecomment-3354141)
solved an issue I was having on OS X 10.6, where running `virtualenv .` locally
failed with this error:

```
install_name_tool: for architecture cputype (16777223) cpusubtype (-2147483645)
Could not call install_name_tool -- you must have Apple's development tools installed
```

The fix was installing the `feature/install_name_tool` branch of Gregg Lind's
virtualenv fork.

```
git clone https://github.com/gregglind/virtualenv.git
cd virtualenv
git checkout feature/install_name_tool
sudo python setup.py install
```

Thank goodness (and Gregg) I don't have to download and reinstall XCode.





