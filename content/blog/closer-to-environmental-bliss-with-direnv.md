---
type: post
title: "Closer to Environmental Bliss with direnv"
date: 2016-03-07 16:10:55 -0500
comments: true
tags:
---

I was a long time user of [RVM](https://rvm.io/) for installing and switching
Rubies. It made my life pretty easy even as I listened to others struggle with
it. I was a little uncomfortable putting other things like project-specific
environment variables in my `.rvmrc` files. It seemed dirty, but it worked and I
rolled with it for many years.

Eventually, I tried something different. I'd been sold on the idea of using
multiple, simpler tools together. Here's the result.

<!--more-->

## Installing a Ruby: `ruby-install`

{{< highlight text >}}
> brew install ruby-install
> ruby-install ruby-2.2
{{< / highlight >}}

Got me the Ruby I wanted. "Too simple," I thought. "Now I won't be able to say I want to use that version."

## Use a Ruby: `chruby`

{{< highlight text >}}
> brew install chruby
> chruby
   ruby-2.2.4
{{< / highlight >}}

And then `chruby` spat out a list of installed Rubies.

OK.

{{< highlight text >}}
> chruby ruby-2.2.4
> chruby
 * ruby-2.2.4
> ruby --version
ruby 2.2.4p230 (2015-12-16 revision 53155) [x86_64-darwin15]
{{< / highlight >}}

"Hunh," thinks I. `chruby` will by default find Rubies installed in the default
directory used by `ruby-install`: `/opt/rubies/` and `~/.rubies/`.

"OK. But now I have to run that for each project to get the Ruby the project
needs."

## Project-Specific Environments: `direnv`

Enter `direnv`.

{{< highlight text >}}
> brew install direnv
{{< / highlight >}}

Then create an `.envrc` file in a project root directory with the appropriate
commands for the project. It can be made pretty convenient by adding the
following function to a `.direnvrc` file in your HOME directory.

{{< highlight bash >}}

# add to ~/.direnvrc
use_ruby() {
  # enable the chruby command in an environment
  source /usr/local/opt/chruby/share/chruby/chruby.sh

  # desired Ruby version as first parameter
  local ver=$1

  # if version not given as parameter and there is a .ruby-version file, get
  # version from the file
  if [[ -z $ver ]] && [[ -f .ruby-version ]]; then
    ver=$(cat .ruby-version)
  fi

  # if the version still isn't set, error cause we don't know what to do
  if [[ -z $ver ]]; then
    echo Unknown ruby version
    exit 1
  fi

  # switch to the desired ruby version
  chruby $ver

  # Sets the GEM_HOME environment variable to `$PWD/.direnv/ruby/RUBY_VERSION`.
  # This forces the installation of any gems into the project’s sub-folder. If
  # you’re using bundler it will create wrapper programs that can be invoked
  # directly instead of using the `bundle exec` prefix.
  layout_ruby
}

{{< / highlight >}}

Thanks to [Steve Tooke](http://tooky.co.uk/using-direnv-and-chruby-together/)
for most of the above.

So one of my projects is a Rails app. It's `.envrc` looks like:

{{< highlight bash >}}
# .envrc for a Rails project
export RUBY_GC_MALLOC_LIMIT=90000000
export RUBY_GC_HEAP_FREE_SLOTS=200000

use ruby 2.2.4
{{< / highlight >}}

This sets some environment variables suitable for testing and switches my Ruby
environment around. I get Ruby version 2.2.4 and GEM_HOME, GEM_PATH, and
GEM_ROOT adjusted to bring in global gems from the 2.2.4 install and
project-specific gems from `<project-root>/.direnv/ruby` thanks to direnv's
`layout ruby` [feature](https://direnv.readthedocs.org/en/latest/commands/direnv-stdlib/#layout-ruby).
I recommend adding `.direnv` to your project's `.gitignore`.

## Bonus Round - Chef DK!

I use Chef and the recommended way to install Chef on a development workstation
is to [install the Chef DK](https://downloads.chef.io/chef-dk/). Chef DK
includes its own embedded Ruby and gem environment and `chruby` knows nothing
about it. We can use `direnv` to twiddle the environment for Chef projects.

{{< highlight bash >}}

# add to ~/.direnvrc
use_chefdk() {
  EXPANDED_HOME=`expand_path ~`

  # Override the GEM environment

  log_status "Overriding default Ruby environment to use ChefDK"
  RUBY_ABI_VERSION=`ls /opt/chefdk/embedded/lib/ruby/gems/`
  export GEM_ROOT="/opt/chefdk/embedded/lib/ruby/gems/$RUBY_ABI_VERSION"
  export GEM_HOME="$EXPANDED_HOME/.chefdk/gem/ruby/$RUBY_ABI_VERSION"
  export GEM_PATH="$EXPANDED_HOME/.chefdk/gem/ruby/$RUBY_ABI_VERSION:/opt/chefdk/embedded/lib/ruby/gems/$RUBY_ABI_VERSION"

  # Ensure ChefDK and its embedded tools are first in the PATH

  log_status "Ensuring ChefDK and it's embedded tools are first in the PATH"

  PATH_add $EXPANDED_HOME/.chefdk/gem/ruby/$RUBY_ABI_VERSION/bin/
  PATH_add /opt/chefdk/embedded/bin
  PATH_add /opt/chefdk/bin
}

{{< / highlight >}}

Thanks to [Seth Chisamore](https://github.com/schisamo) for the Chef DK
function.

Now my `.envrc` files in Chef projects have a line like this:

{{< highlight bash >}}
# .envrc in a Chef project
use chefdk
{{< / highlight >}}

It's working for now and I can happily switch between Chef DK projects and
projects needing their own Rubies (e.g. Rails sites).
