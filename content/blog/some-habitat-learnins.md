---
type: post
title: "Some Habitat Learnins"
date: 2016-10-29 14:30:28 -0700
published: false
comments: true
tags:
---

`$(hab pkg path <origin>/<thingie>)` in runtime hooks is a code smell.
(Deployment smell?) Strongly consider using `$(pkg_path_for <origin>/<thingie>)`
in your plan to evaluate paths to dependencies at build time, then write that
stuff out into your hooks or scripts for your hooks to call.

<!--more-->

`$(hab pkg path <origin>/<thingie>)` at runtime does not allow the Habitat
automation to make promises about using only the dependencies declared in the
plan at the versions used at build time. It will return the path to the latest
version of `<origin>/<thingie>` on the host which might be a later version that
was installed to meet a requirement of a different Habitat package installed on
the host.

One way to fix this is to produce a script at build time with paths to things exported as environment variables.

{{< highlight bash >}}

{{< / highlight >}}

Ooo. Which brings up setting build-time paths and run-time paths. I've
experimented during build to write out a file of paths to source at run-time
with values using the build-time-computed paths to dependencies. That way, I'm
sure I've sourced the version of a dependency that I built against.

[12:54]
https://github.com/chef/supermarket/blob/42c32a6411b1569ba23499800d42d24cae2c080a/habitat/supermarket/plan.sh#L128-L137

And then the hooks start with `. {{pkg.path}}/place/you/dropped/the/runtime_environment.sh`

[12:57]
@smith The run hook is the place to _source_ it, but I'm of the opinion that build-time is the place to _compute_ the paths. `$(pkg_path_for bundler)` during build is going to source the version I've built against. `$(hab pkg path core/bundler)` during runtime hook is not guaranteed to compute to the version of the dependency I built against.
