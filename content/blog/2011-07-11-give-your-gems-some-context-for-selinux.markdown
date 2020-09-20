---
layout: post
title: "Give Your Gems Some Context for SELinux"
date: 2011-07-11 18:55
comments: true
categories: [RHEL, CentOS, Linux, SELinux]
---

Are you deploying your Ruby app to Red Hat or CentOS? Is Passenger complaining about not having permission to load the gems you have bundled with your app--e.g. failed to map segment from shared object: Permission denied--despite all the basic filesystem permissions looking correct? Would you like to avoid the nearly ubiquitous advice to disable SELinux, thus turning off the security that comes out of the box with SELinux enabled?

```
sudo chcon -R -h -t httpd_sys_content_t <gems path>
```

* `chcon` :: Changes the security context for those executables and libraries.
* `-R` :: (recursive) All the files here, please.
* `-h` :: Affect symbolic links instead of any referenced file.
* `-t httpd_sys_content_t` :: Set the TYPE in the security context.

We deploy with capistrano, so I have added this as a task called after 
`bundle:install` vendors my gems to `<appdir>/shared/bundle`.

```ruby
namespace :application do
  after 'bundle:install', 'application:update_selinux'
  task :update_selinux, :roles => :web do
    puts 'Updating SELinux context for bundled gems'
    run "#{try_sudo} chcon -R -h -t httpd_sys_content_t #{applicationdir}/shared/bundle/"
  end
end
```
