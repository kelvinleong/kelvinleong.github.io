---
layout: post
title:  "Jekyll 3.0 Upgration"
date:   2016-03-09 15:24:05
categories: Theme
---

> GitHub Pages is now running the latest major version of Jekyll, Jekyll 3.0, and with it, many of the complexities associated with publishing have been further simplified, meaning it's now easier and faster to publish beautiful sites for you and your projects.

My github page is built on top of Jekyll 2.0, which runs well on my local testing while throwing out "404" errors while visited from the github domain. And the problems is due to github pages updating Jekyll to version 3.0 while my page is still running based on v2.0.

To solve the problem, what I have to do is simply upgrading the Jekyll version and re-deployed my repository.

Here are steps to go:

* **Update your ruby version to 2.0**

Jekylly 3.0 require ruby-2.0 environment. The quickest way of installing Ruby on Rails with rvm is to run the following commands as a regular user:

```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable --rails
```

Following a long installation procedure, all you need to do is source the rvm scripts by typing:

```
source ~/.rvm/scripts/rvm
rvm list / rvm list known
rvm use ruby_version(ruby-2.0)
```

* **Update your Jekyll**

Well, just use the following command :

```
$ gem update jekyll
```

* **Revise Jekyll configuration**

As announced, github pages will no longer support "redcarpet" markdown and "pygment" highlighter and change to "Kramdown" and "Rouge" respectively. Therefore, change the mardown and highlighter value in "\_config.yaml" file to continue.

```
kramdown:
  input: GFM
  syntax_highlighter: rouge
paginate: 7
```

In Jekyll 3 and above, relative permalinks have been deprecated. Removing the following line from your \_config.yml file:
```
relative_permalinks: true
```

Besides that, I got the following problems that "Deprecation: You appear to have pagination turned on, but you haven't included the jekyll-paginate gem." Ensure you have "gems: [jekyll-paginate, jekyll-gist]" in the \_config.yaml file. As well, ensure you have jekyll-paginate and jekyll-gist installed using the following command:

```
$ gem install jekyll-paginate
$ gem install jekyll-gist
```

* **Enable remote access Jekyll page thru local network**

Try jekyll serve --host=0.0.0.0 when you invoke Jekyll on the command line.

That will make Jekyll's HTTP server bind to all available IPs, rather than just to localhost.

You can also add this to your \_config.yml with host: 0.0.0.0. GitHub will simply ignore this when you push, so it's safe to use if you don't mind having your work openly accessible on your network.

Without ***--host=0.0.0.0 Jekyll*** will output something like this when you start up:

```
$ jekyll serve
[...]
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.
```

But with ***--host=0.0.0.0*** (or host: 0.0.0.0 in \_config.yml) you'll notice that it's listening on all interfaces (represented by 0.0.0.0) rather than just listening on the loopback interface (represented by 127.0.0.1)

```
$ jekyll serve --host=0.0.0.0
[...]
Server address: http://0.0.0.0:4000/
Server running... press ctrl-c to stop.
```

Reference:

> <https://jekyllrb.com/docs/upgrading/2-to-3/>
