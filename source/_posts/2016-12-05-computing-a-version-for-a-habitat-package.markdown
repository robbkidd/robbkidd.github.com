---
layout: post
title: "Computing a Version for a Habitat Package"
date: 2016-12-05 09:53:44 -0500
comments: true
categories: habitat
---

Sometimes when packaging a thing in [Habitat](https://www.habitat.sh/), the plan really should not have the version hard-coded if the plan is packaging something from which the version could be computed.

<!-- more -->

<iframe src="//giphy.com/embed/13RBYnxA3RBpOU?html5=true" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/realitytvgifs-lady-gaga-work-13RBYnxA3RBpOU">via GIPHY</a></p>

Take for example [the core-plan for cacerts](https://github.com/habitat-sh/core-plans/blob/master/cacerts/plan.sh). The plan downloads the latest cacerts and figures out what the specific version is from the "source" downloaded by calling a custom `update_pkg_version()` function in the plan right after download (the earliest possible moment to figure the version out). Here's what that function looks like today:

```bash cacert version look up
update_pkg_version() {
  # Extract the build date of the certificates file
  local build_date=$(cat $HAB_CACHE_SRC_PATH/$pkg_filename \
    | grep 'Certificate data from Mozilla' \
    | sed 's/^## Certificate data from Mozilla as of: //')

  # Update the `$pkg_version` value with the build date
  pkg_version=$(date --date="$build_date" "+%Y.%m.%d")
  build_line "Version updated to $pkg_version from CA Certs file"

  # Several metadata values get their defaults from the value of `$pkg_version`
  # so we must update these as well
  pkg_dirname=${pkg_name}-${pkg_version}
  pkg_prefix=$HAB_PKG_PATH/${pkg_origin}/${pkg_name}/${pkg_version}/${pkg_release}
  pkg_artifact="$HAB_CACHE_ARTIFACT_PATH/${pkg_origin}-${pkg_name}-${pkg_version}-${pkg_release}-${pkg_target}.${_artifact_ext}"
}
```

Cool. "Just figure it out, Plan."

That's the case for when a Habitat plan is packaging something downloaded from elsewhere. Another case is when Habitat is being used to package a Thing and the plan co-exists within the Thing's codebase. I'm working through packaging an existing Rails application with Habitat. The app and the plan co-exist in a single git repository. The application already has a mechanism to declare its version using git tags. We can tell the plan to just figure it out in this case, too.

```bash app in git repo: version lookup
pkg_build_deps=(
  ...
  core/git
  ...
)

determine_version() {
  pkg_version=$(git describe)
  pkg_dirname=${pkg_name}-${pkg_version}
  pkg_filename=${pkg_dirname}.tar.gz
  pkg_prefix=$HAB_PKG_PATH/${pkg_origin}/${pkg_name}/${pkg_version}/${pkg_release}
  pkg_artifact="$HAB_CACHE_ARTIFACT_PATH/${pkg_origin}-${pkg_name}-${pkg_version}-${pkg_release}-${pkg_target}.${_artifact_ext}"
}
```

`core/git` needs to be included in the plan's build dependencies and then we can use `git describe` to get the tag for the current commit (or nearest tag and how far away a commit is from it). This function needs to set one more metadata value than in cacerts—`pkg_filename`—which is based on the computed version. The plan is _in_ the source repository, so we don't need to download anything. The Habitat `do_download()` and `do_verify()` functions get overridden with:

```bash app in git repo: fake out download
do_download() {
  determine_version

  build_line "Fake download! Creating archive of latest repository commit."
  # source is in this repo, so we're going to create an archive from the
  # appropriate path within the repo for the rest of the plan callback chain
  # to pick up on
  cd $PLAN_CONTEXT/../..
  git archive --prefix=${pkg_name}-${pkg_version}/ --output=$HAB_CACHE_SRC_PATH/${pkg_filename} HEAD src/
}

do_verify() {
  build_line "Skipping checksum verification on the archive we just created."
  return 0
}
```

We use `git archive` to produce a tarball of the application source that follows the conventions of the default `do_download()` function. There's nothing to verify about it because we just created it. The default `do_unpack()` is left to run without being overridden because `do_download()` produced a conventional source tarball.

With this in place, the existing process for version bumping the application remains unchanged and does not require any hard-coding in metadata.

<iframe src="//giphy.com/embed/l3nWqD4ViFej9REAw?html5=true" width="480" height="269" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/laughing-laugh-bill-nye-l3nWqD4ViFej9REAw">via GIPHY</a></p>
