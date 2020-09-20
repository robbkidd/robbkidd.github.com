---
layout: post
title: "One Unity Launcher"
date: 2012-04-05 14:43
comments: true
categories: [Ubuntu, Linux]
---

T'was decided that [the default behavior for Unity's launcher will be to appear only on the primary monitor in Ubuntu 12.04](http://www.omgubuntu.co.uk/2012/03/ubuntu-12-04-multi-monitor-to-be/).
For some reason, when I upgraded to 12.04 (beta) today, my setting was
for the launcher to appear on ALL desktops. Not a fan of it on all desktops.

For those who want to change this setting, you'll need to install the Compiz
configuration manager.

```
sudo aptitude install compizconfig-settings-manager
```

Once installed, run it.

1. Hover the Dash Home (Ubuntu icon on launcher)
1. Type in `compiz` and click on the `CompizConfig Settings Manager` icon that appears.
1. Select `Desktop` on the left.
1. Click on `Ubuntu Unity Plugin` on the right.
1. Select the `Experimental` tab and scroll all the way down.

The setting for where launchers appear is `Launcher Monitors`. As of
 today (2012/04/05), you can select either `Primary Desktop` or `All Desktops`.
