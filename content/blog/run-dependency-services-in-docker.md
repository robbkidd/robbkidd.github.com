---
type: post
title: "Run Dependency Services in Docker"
date: 2016-03-10 08:47:17 -0500
comments: true
tags:
---

My current primary project is a Rails app that depends on PostgreSQL and Redis.
I used to run those services directly on my Mac OS development workstation, but
didn't like the untidiness of having services I wasn't using all the time
running all the time. Or that the default install might leave the services
listening on all interfaces so that everyone in the coffee shop could poke at my
test data. (Nah. I've got a firewall and [so should you](https://www.obdev.at/products/littlesnitch).)
I figured, "Hey, I could use Docker to cordon these services off."

<!--more-->

![Docker, so hot right now](/images/so-hot-right-now.jpg)

# Prepare Your App

First, I had to make sure my config files would be happy with the usual defaults
of services running on localhost.

## PostgreSQL

For PostgreSQL, we update the database configuration to accept an IP address
from an environment variable, otherwise use localhost as it did before.

{{< highlight yaml >}}
# config/database.yml
development:
  adapter: postgresql
  database: derelicte_development
  host: <%= ENV['POSTGRES_IP'] || 'localhost' %>
  pool: 30

test:
  adapter: postgresql
  database: derelicte_test
  host: <%= ENV['POSTGRES_IP'] || 'localhost' %>
{{< / highlight >}}

NOTE: We will still need a version of PostgreSQL installed—but not services
*running*—on the local filesystem for development libraries to be available to
be able to build the `pg` gem. Configure bundler to know where to find
PostgreSQL libraries. For example, we have
[PostgreSQL installed via Homebrew](https://www.codefellows.org/blog/three-battle-tested-ways-to-install-postgresql),
so we run the following to configure bundler:

{{< highlight text >}}
$ bundle config build.pg --with-pg-config=/usr/local/bin/pg_config
{{< / highlight >}}

## Redis

Our application had long received the Redis connection information via an
environment variable in an initializer.

{{< highlight ruby >}}
# config/initializers/redis.rb
# ...
  redis_url = ENV['REDIS_URL'] || 'redis://localhost:6379/0/derelicte'
# ...
{{< / highlight >}}

# Services in Docker

With the application configurable by environment variables (but with sensible
defaults), we can move on to installing Docker and getting the services running.

## Docker Prerequisites

Docker on a Mac requires some sort of hypervisor—like Virtualbox or VMware—to
run a Linux virtual machine (VM) which supports the container features Docker
uses. Virtualbox is the easiest to get started with.

* [Download](https://www.virtualbox.org/wiki/Downloads) and install Virtualbox.

* Install `docker-machine` and `docker`. With [Homebrew](http://brew.sh/):

{{< highlight text >}}
# install the things
$ brew install docker docker-machine
{{< / highlight >}}

* Create a `docker-machine` VM on which to run Docker services. The following command assumes your hypervisor is VirtualBox and that you will name this host "default".

{{< highlight text >}}
# create the docker-machine VM
$ docker-machine create --driver virtualbox default
{{< / highlight >}}

* Set the environment variables for the application configuration. This can be
done by hand or we can automate the setup with something like `direnv`. See
[Closer to Environmental Bliss with Direnv](/blog/2016/03/07/closer-to-environmental-bliss-with-direnv) for more on direnv.

{{< highlight bash >}}
# environment setup
eval $(docker-machine env default)
export DOCKER_IP=$(docker-machine ip default)
export POSTGRES_IP=$DOCKER_IP
export REDIS_URL="redis://${DOCKER_IP}:6379/0/derelicte"
{{< / highlight >}}

## Manage the Services

Now we can use `docker` to control the Docker services running in the `default`
Docker Machine VM. Now we can actually run PostgreSQL and Redis.

{{< highlight text >}}
# test docker command
$ docker info
{{< / highlight >}}

That should return lots of information about the running Docker services in the
VM.

{{< highlight text >}}
# start postgresql
$ docker run \
      --detach=true \
      --env POSTGRES_USER=$USER  \
      --publish 5432:5432 \
      --name derelicte-pg \
      postgres:9.3
{{< / highlight >}}

{{< highlight text >}}
# start redis
$ docker run \
      --detach=true \
      --publish 6379:6379 \
      --name derelicte-redis \
      redis:3.0
{{< / highlight >}}

These `docker run` commands retrieve already configured container images from
DockerHub and runs them. Ports are forwarded from the running containers to the
Docker Machine VM's network interface. The commands are a bit long, so I
recommend scripting them, possibly by adding them to a rake task like
`setup:docker`. Maybe go a step further and have a rake task that chains the
docker runs with `db:create`, `db:migrate`, `db:test:prepare`.

{{< highlight text >}}
# check on running containers
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
9ca2dbac647e        redis:3.0           "/entrypoint.sh redis"   4 seconds ago       Up 3 seconds        0.0.0.0:6379->6379/tcp   derelicte-redis
5009b1db0c08        postgres:9.3        "/docker-entrypoint.s"   4 seconds ago       Up 3 seconds        0.0.0.0:5432->5432/tcp   derelicte-pg
{{< / highlight >}}

At this point, with the application configured and the services running, we can
start running tests and a development server on the workstation. PostgreSQL and
Redis are safely listening on a host-only interface on the Docker Machine VM. To
scrap the services and start fresh, stop the containers individually.

{{< highlight text >}}
# blow away the database and redis containers - WILL LOSE DATA
$ docker rm -f derelicte-redis
$ docker rm -f derelicte-pg
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
{{< / highlight >}}

Then, if those run commands are in a rake task:

{{< highlight text >}}
# run those containers again
$ time be rake setup:docker
# lotsa output
8fb8df60a4aea74d84c87be8a16b10315c8d269e5c44b34df74ff53c76738357
d5f363fe1940fe39257c7a5f488d1c6007644c24039f091b634d5857ed69ab34
bundle exec rake setup:docker  2.61s user 0.93s system 91% cpu 3.868 total

$ time be rake db:create db:migrate db:seed db:test:prepare
# a lot more output
bundle exec rake db:create db:migrate db:seed db:test:prepare  4.41s user 1.64s system 73% cpu 8.175 total
{{< / highlight >}}

Aww yeah, 4 second recreate and 8 second setup and we're ready to test.

![It's beautiful!](/images/beautiful.gif)

We can blow away the entire Docker Machine VM for even more pave-the-earth
destruction, but this will destroy *all* the containers in that VM and will
lengthen the next startup because images will need to be downloaded from
DockerHub again. Beware if using the same Docker Machine for more than
one project.

{{< highlight text >}}
# destroy and recreate the docker machine
$ docker-machine rm default
About to remove default
Are you sure? (y/n): y
Successfully removed default

$ docker-machine create -d virtualbox default
{{< / highlight >}}

We should reinitialize the environment variables set earlier—super handy if you
control them with direnv: `direnv reload`—because the docker-machine IP may have
changed after being recreated.
