---
layout: post
title:  "Using jekyll with docker for windows"
date:   2017-03-13 20:00:00 +0000
categories: jekyll docker
excerpt_separator: <!--more-->
---
Windows is not offical supported by jekyll, so docker is an easy way to install, update and run jekyll on a windows machine. The following description requires a folder with the jekyll directory structure.
<!--more-->

Building the container
----------------------
To start with jekyll a image based on the offical ruby docker image is build.
{% highlight docker %}
FROM ruby:2.4.0
MAINTAINER test <test@mail.net>
{% endhighlight %}

Expose the port 4000 used by default by the jekyll `serve` option.
{% highlight docker %}
EXPOSE 4000
{% endhighlight %}

Setting the working dir.
{% highlight docker %}
WORKDIR /srv/jekyll
{% endhighlight %}

Overriding the `$BUNDLE_PATH` environment variable to let bundler install the gems in a directory that can be mapped as a volume, so installed gemes will be reused at container start and not be reinstalled while container startup.
{% highlight docker %}
ENV BUNDLE_PATH /box
{% endhighlight %}

Finally `bundler` will install the `gems` from the `gemfile` and jekyll is startet.
{% highlight docker %}
ENTRYPOINT bundler install && jekyll serve --watch --force_polling --drafts
{% endhighlight %}

Due the fact docker for windows does not support `inotify`, the `--watch` option does not work, so the additional option `--force_polling` is required to automatically regenerate the site.

Starting the container
----------------------
To make building and starting the container more easy a docker-compose via `docker-compose up` is used.
{% highlight YAML %}
version: '3'
services:
  jekyll-serve-blog:
    build: .
    volumes:
      - c:/docker/jekyll/blog:/srv/jekyll
      - data:/box
    ports:
      - "4000:4000"
volumes:
  data:
{% endhighlight %}

Environment
-----------
The used jekyll version is 3.3.1 and the output of `docker version` is:
{% highlight shell %}
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   60ccb22
 Built:        Thu Feb 23 10:40:59 2017
 OS/Arch:      windows/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
 Experimental: true
{% endhighlight %}

Links
-----
[inotify does not work with docker deamon in a virtual machine](https://github.com/docker/docker/issues/18246)  
[cache bundle install with docker](https://medium.com/@fbzga/how-to-cache-bundle-install-with-docker-7bed453a5800#.i1ysdy6u5)
