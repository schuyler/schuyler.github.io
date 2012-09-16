---
layout: post
title: "How to make an AMI from a running Ubuntu EC2 instance."
date: 2012-09-16 09:34
comments: true
categories: ec2
---

First, you'll need the `ec2-ami-tools` package. Assuming you're running Ubuntu
12.04, go into `/etc/apt/sources.list` and uncomment the `multiverse` lines.
Then run:

``` bash
apt-get update
apt-get install -y ec2-ami-tools
```

Next, upload your `pk-*.pem` and `cert-*.pem` files to `/mnt` on the
running instance. Don't worry, `ec2-bundle-vol` ignores `*.pem` and the
entirety of `/mnt` by default, so your credentials won't get built into
the image.

Now, as root, create a place to put the image bundle and then build it.

``` bash
mkdir /mnt/bundle
ec2-bundle-vol -d /mnt/bundle -k pk-*.pem -c cert-*.pem \
               -u YOUR_AWS_ID_WITHOUT_DASHES -r x86_64
```

This will bundle up a 64-bit image in `/mnt/ami-instance-name`. Change `x86_64`
to `i386` if you're building a 32-bit image instead. The process took
about 2.5 minutes on an m2.4xlarge instance.

Next, upload the AMI bundle to S3. The `-b` argument to `ec2-upload-bundle`
lets you specify which S3 bucket to upload to. If the argument includes a
slash, the part after is an optional AMI name. The name doesn't mean anything
to Amazon, but it lets you keep multiple bundles in the same bucket.

``` bash
ec2-upload-bundle -b my-ec2-bucket/my-ami \
    -m /mnt/bundle/image.manifest.xml
    -a YOUR_AWS_ACCESS_KEY -s YOUR_AWS_SECRET_KEY
```

This process also takes a couple minutes.

Finally, register your AMI with EC2.

``` bash
ec2-register -K pk-*.pem -C cert-*.pem my-ec2-bucket/my-ami/image.manifest.xml
```

EC2 should respond with something like:

```
IMAGE   ami-deadbeef
```

Voilà, you are are good to go. If you wind up doing this a lot, consider
scripting the process [like Alistair Davidson did](http://instantbadger.blogspot.com/2009/09/how-to-create-and-save-ami-image-from.html).
