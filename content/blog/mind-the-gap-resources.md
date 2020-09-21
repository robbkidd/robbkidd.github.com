---
type: post
title: "Mind the Gap: Resources"
date: 2014-04-19 12:20:00 -0400
comments: true
tags: [chef]
---

I spoke this week at [ChefConf](http://chefconf.opscode.com/chefconf/)
about how we use Chef at [my company](http://www.lgscout.com) to produce
an installer for our on-premises product.
[Slides for the talk](https://speakerdeck.com/robbkidd/mind-the-gap) are
up, but not very useful without my rambling that went with them. Instead of
making attendees madly write down the resources slide, here are links to the
resources and tools I mentioned in the talk.

<!--more-->

* [Vagrant](http://vagrantup.com) from [Mitchell Hashimoto](http://about.me/mitchellh)
  and Hashicorp. We use Vagrant in our chef repo to spin up a VM for each of our node
  types and perform chef runs to populate caches of things retrieved from
  the network.
* [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier) by [Fabio Rehm](http://fabiorehm.com/)
  is used for centralizing the node's Chef cache into a single shared
  folder located in the chef repo. We do not use it for caching APT
  packages.
* [apt-cacher-ng](https://www.unix-ag.uni-kl.de/~bloch/acng/) (ACNG) is what we
  use for caching APT packages. On one node at chef compile time, ACNG
  is installed and configured. All nodes are configured to `apt-get` with
  the ACNG node as their proxy. After a successful convergence on all
  nodes, the ACNG cache of downloaded packages is copied into the chef
  repo.
* [fpm](https://github.com/jordansissel/fpm) by [Jordan Sissel](http://www.semicomplete.com/)
  is the wonderment with which we package our own components and package
  up the chef repo with a chef installer and the populated caches. Super
  time saver for producing a variety of package formats.
* [knife-solo](https://matschaffer.github.io/knife-solo/) by [Mat Schaffer](http://matschaffer.com/)
  was not explicitly mentioned in the talk, but gets a shout out at the
  end and here. `knife-solo` has made our lives easier by letting us get
  our chef repo up onto test and staging servers and starting a chef run
  is out chef-solo-oriented world.
