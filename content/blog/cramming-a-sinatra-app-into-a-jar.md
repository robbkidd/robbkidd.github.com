---
type: post
title: "Cramming a Sinatra App into a JAR"
date: 2015-05-06 14:36:54 -0400
comments: true
published: false
tags: [Ruby, JRuby, Sinatra, Warbler, Java]
---

We recently solved a problem for allowing our internal users to easily update
records in a legacy data store by providing a simple web interface via a Sinatra
app. Sweet! Now we have to deploy it: Ruby runtime in production. Oh. Nuts. How?

<!--more-->

The new hotness would be to generate a Docker image of the app, but the
organization is still considering the security implications of Docker containers
in production. Deploy to a Linux host and use the distribution's Ruby package?
No, version is too old. Chef or Puppet to install an up-to-date MRI Ruby runtime
on the host? Maybe, but we like to put as many of the dependencies into a
component package as possible. Omnibus? On the to-learn list and our users would
really like to use this interface soon. What one, trustworthy thing could I
easily put on the server to let it run a Sinatra app?

Java.

Oh yeah. JRuby.

An introduction to a fake version of the app might be in order for me to show
the work.

The libraries used in the app and to package the app as a JAR file.

{{< highlight ruby >}}
# Gemfile
source "https://rubygems.org"

gem "haml"      # It's a simple app, yo.
gem "sinatra"   # Sing for me, Frank.
gem "puma"      # JRuby-friendly app server

group :development do
  gem 'pry'     # prettier irb
  gem 'warbler' # heavy lifter for JAR'ing the app
end
{{< / highlight >}}

The core of the app itself.

{{< highlight ruby >}}
# lib/awesome_sauce.rb
require 'sinatra'

class AwesomeSauce < Sinatra::Application
  require 'haml'

  get '/' do
    haml :root_sauciness
  end
end
{{< / highlight >}}

The view for `:root_sauciness`.

{{< highlight ruby >}}
# views/root_sauciness.rb
!!! XML
!!!
%html
 %head
  %title Awesome Sauce
  %meta{ 'http-equiv' => 'Content-Type', :content => 'text/html; charset=utf-8' }
 %body
 %h1 The root of the sauce is awesome.
{{< / highlight >}}

A rackup file. Why not `config.ru`? Because by putting it in `bin/`, Warbler
will create a JAR, not a WAR, and will by default use this file as the first
thing run by the JAR.

{{< highlight ruby >}}
# bin/web.ru
#!/usr/bin/env jruby

$LOAD_PATH.unshift(File.expand_path("#{File.dirname(__FILE__)}/../lib"))
require 'puma'
require 'awesome_sauce'

configure do
  set :views, './views' # foreshadowing: this line will change later
  set :server, %w[puma]
end

AwesomeSauce.run!
{{< / highlight >}}

A Warbler configuration for making the JAR.

{{< highlight ruby >}}
# config/warble.rb
Warbler::Config.new do |config|
  config.features = %w(executable)
  config.dirs = %w(bin lib views)
end
{{< / highlight >}}

OK. Let's run this ...

{{< highlight text >}}
> bundle install
Using rake 10.4.2
Using coderay 1.1.0
Using ffi 1.9.8
Using tilt 2.0.1
Using haml 4.0.6
Using jruby-jars 1.7.20
Using jruby-rack 1.1.18
Using method_source 0.8.2
Using slop 3.6.0
Using spoon 0.0.4
Using pry 0.10.1
Using rack 1.6.1
Using puma 2.11.2
Using rack-protection 1.5.3
Using rubyzip 1.1.7
Using sinatra 1.4.6
Using warbler 1.4.7
Using bundler 1.9.4
Bundle complete! 5 Gemfile dependencies, 18 gems now installed.
Bundled gems are installed into ./vendor.

> bundle exec warble
No executable matching config.jar_name found, using bin/web.ru
rm -f awesome_sauce.jar
Creating awesome_sauce.jar

> java -jar awesome_sauce.jar
Puma 2.11.2 starting...
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:4567
== Sinatra (v1.4.6) has taken the stage on 4567 for development with backup from Puma

{{< / highlight >}}

Opening `localhost:4567` in a browser shows me something like:

## The root of the sauce is awesome.

Sweet.

Let's put that JAR file up on a server (in this case a Vagrant machine
for testing) and try there.

{{< highlight text >}}
# Running from /vagrant
vagrant@vagrant-ubuntu-trusty-64:/vagrant$ java -jar awesome_sauce.jar &
[1] 1588
vagrant@vagrant-ubuntu-trusty-64:/vagrant$ Puma 2.11.2 starting...
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:4567
== Sinatra (v1.4.6) has taken the stage on 4567 for development with backup from Puma

vagrant@vagrant-ubuntu-trusty-64:/vagrant$ curl localhost:4567
<!DOCTYPE html>
<html>
  <head>
    <title>Awesome Sauce</title>
    <meta content='text/html; charset=utf-8' http-equiv='Content-Type'>
  </head>
  <body>
    <h1>The root of the sauce is awesome.</h1>
  </body>
</html>
127.0.0.1 - - [09/May/2015:15:24:36 +0000] "GET / HTTP/1.1" 200 221 0.1170
{{< / highlight >}}

Cool. It works and I only had to install a Java runtime on the server.
Let's try running it from a more production-like directory.

:
