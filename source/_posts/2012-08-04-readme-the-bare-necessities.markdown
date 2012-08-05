---
layout: post
title: "README: the Bare Necessities"
date: 2012-08-04 19:35
comments: true
categories: 
---

I gave a lightning talk at Steel City Ruby Conf 2012 today on writing a
decent README. Instead of putting the slides up somewhere, I thought I
would write up a more detailed post of the talk's points.

<!-- more -->

## Your README should ...

### ... explain why your code exists

Tell me why you wrote this code. Give me a vision statement or an
elevator pitch. "Unlike all the other code on the internet, my project
is awesome." Then follow up with how your project scratches an itch
differently than others.

### ... demonstrate how to install and use the code

It could be as simple as:

```
gem install awesome_tool
```

```
require 'awesome_tool'

awesome_tool do
  awesome_stuff
end
```

If things get complicated, use subsections or wiki pages, but try to
provide steps and examples in the README and link to the complicated
stuff.

As far as examples of use are concerned, start with the most common and
valuable example of using your tool. Help me understand how to make my
life better with your awesomeness.

### ... declare copyright and licensing

Tell me your licensing terms. Can I copy your code? Can I distribute it?
Can I modify it?

### ... describe how to contribute

If I can modify it, how do I get those modifications back to you, the
maintainer? Do you expect code to follow certain conventions? Do you
expect to have tests submitted with the patch? (In general, the answer
here is "yes.")

Provide instructions on setting up the tool's development environment.
Tell me how to run your tests. There is nothing like trying to document
this sort of thing to cause you to throw up your arms and automate this
stuff which only makes your code more awesome.

### ... give credit

Take credit! You wrote an awesome tool! Give credit to contributors.
This encourages others to contribute more awesomeness. List
authors/contributors, link to a CONTRIBUTORS file or punt to GitHub and
link to the contributors graph.

### ... be plain text

Write your README in plain text. We've got decades of convention that
READMEs are readable on a simple terminal with your PAGER of choice.
Don't fight that.

I would go farther and recommend you [write your README in Markdown](https://raw.github.com/robbkidd/activerecord-netezza-adapter/master/README.md). It
is a simple plain text markup that is refreshingly free of noisy markup
like HTML tags that I have to ignore. The benefit is that some tools 
like GitHub and GitLabHQ 
[turn your README.md into a web page](https://github.com/robbkidd/activerecord-netezza-adapter/blob/master/README.md).
Yay! Your project has a README *and* a web page with structure,
hyperlinks and syntax-highlighted code blocks.

Even better, a tool called [PanDoc](http://johnmacfarlane.net/pandoc/)
can turn your README.md into a PDF. If you provided headings and
sub-headings (with hash marks "#") in your markdown, you can tell PanDoc
to include a table of contents in your PDF for easy navigation. 

## Leveling Up?

Tim Preston-Werner of GitHub fame has an arguement for [README Driven
Development](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html).
Before you test-drive, before you behavior-drive, document your vision
for what your code will do and write some examples of code you wish you
had. Give that a read and let me know what you think.

## What next?

What sort of documentation you produce after your essential README
depends on what your awesome tool really is, how it is used and what
sort of audience you expect to have for the docs. Library code probably
benefits best from rdoc-style embedded comments that can be compiled
into a library reference. Web APIs can be documented with guide-style
pages providing the lay of the land with references to tests written
specifically as examples of use. 

End-user documentation can be deviously tricky. Simple, but
time-consuming to write at the beginning, end-user docs end up
atrophying, quickly becoming documents filled with misdirection as your
tool grows. I've got a vision of end-user documentation being written in
something like Cucumber to include steps that can capture named
screenshots of the UI state at points in the scenarios. Markdown
documentation--possibly contained in the preamble section of the
Cucumber features--would reference the named screenshots. Run the
documentation Cucumber scenarios, run the markdown through PanDoc and
you've got end-user documentation with up-to-date screenshots. No more
weeks spent manually screenshotting your app while you monkey-push
buttons and enter data to get the app in the right state for the
documentation scenario just because you changed the order of the nav
bar.

I've not cracked this last nut. If you are interested in when I do, let
me know. If *you* have cracked this nut or have some insight into how, I
would love to talk to you. If there is interest, I plan on writing more
here or in another venue where collaboration can occur about techniques
to make documentation less painful. 
