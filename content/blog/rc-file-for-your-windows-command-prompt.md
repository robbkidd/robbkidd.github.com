---
type: post
title: "RC file for your Windows Command Prompt"
date: 2012-07-06 14:41:00 -0400
comments: true
tags: [Windows, Vagrant]
---

I had trouble today with my Windows `%HOME%` changing and throwing off
[Vagrant](http://vagrantup.com)'s awareness of what boxes are installed.
This may be your problem if Vagrant on Windows was working happily for
you and then barks about no boxes.

{{< highlight text >}}
C:\Users\you> vagrant box list
There are no installed boxes! Use `vagrant box add` to add some.

C:\Users\you>dir /b .vagrant.d\boxes
base-i-really-exist
centos-does-too

C:\Users\you>echo %HOME%
<somewhere not C:\Users\you>

C:\Users\you>set HOME=%USERPROFILE%

C:\Users\you>vagrant box list
base-i-really-exist
centos-does-too
{{< / highlight >}}

`%HOME%` was different depending on whether I logged into Windows while
on my Active Directory managed network or while I was offline. When on
the AD network, `%HOME%` is set by policy to a mapped drive. When offline,
`%HOME%` is my Windows `%USERPROFILE%`. I installed Vagrant and some base
boxes while off my AD network and everything worked as expected. Base boxes
were added to `%USERPROFILE%/.vagrant.d/`. Back in the office and logged in
on the AD network, Vagrant was no longer aware of the base boxes because
`%HOME%` now pointed at my personal mapped drive.

Creating a batch file to set `%HOME%` to `%USERPROFILE%` solved the problem.

{{< highlight bat >}}
# cmdrc.bat
REM %USERPROFILE%/cmdrc.bat
@echo off

set HOME=%USERPROFILE%
{{< / highlight >}}

For UNIXy goodness, name it `cmdrc.bat`, place it in `%USERPROFILE%` and then add the following to the registry so that this file is run whenever a command prompt opens.

{{< highlight text >}}
#text cmdrc.reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\Command Processor]
"AutoRun"="%USERPROFILE%\\cmdrc.bat"
{{< / highlight >}}
