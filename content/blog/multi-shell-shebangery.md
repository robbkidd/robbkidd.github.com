---
type: post
title: "Multi-Shell Shebangery"
date: 2019-09-07 19:47:44 -0500
comments: true
tags: [ruby, windows, sh, bash, shell]
---
I have been poking around inside RubyInstaller, the venerable package for installing a Ruby environment on a Windows host. I became entranced by the magical incantation that appears at the top of the Ruby binstubs installed by RubyInstaller. The incantation is what makes these binstubs work in either Windows Command Shell or in UNIXy sh/bash or even processed directly by the Ruby interpreter. It is a marvel, though it took me a while to work out how it does its magic. I thought I would pick it apart here, for my own memory and for anyone else lost wondering what's going on in there.

<!--more-->

Here is the multi-shell shebangish header that appears at the beginning of `bundle.cmd`, `gem.cmd` and so on for the Windows install of Ruby with RubyInstaller.

{{< codecaption lang="text" title="Head of Ruby .cmd files" >}}
:""||{ ""=> %q<-*- ruby -*-
@"%~dp0ruby" -x "%~f0" %*
@exit /b %ERRORLEVEL%
};{ #
bindir="${0%/*}" #
exec "$bindir/ruby" -x "$0" "$@" #
>, #
} #
#!/usr/bin/env ruby
[ ... actual ruby code ... ]
{{< /codecaption >}}

Hieroglyphs. What on earth is going on in here?

## Let's Take an Easy One First

`-*- ruby -*-` is a hint to text editors that this file is Ruby, despite the file extension. It is ignored by all the script interpretors through various means as described below.

## As Windows Command Shell

First, let's interpret it from the perspective of execution from a Windows Command Shell. It looks like BATCH script.

{{< highlight text >}}
:""||{ ""=> %q<-*- ruby -*-
{{< / highlight >}}

`:` is the start of a one-line comment in BATCH. So the rest of this line is ignored. We'll come back to the rest of this line later.

```
@"%~dp0ruby" -x "%~f0" %*
```

This is the first line that the Windows Command Shell will attempt to interpret.
* `@` means don't echo the contents of this line to the terminal. So the user does not see the command, only the command's output.
* `"%~dp0ruby"`:
  - `%0` is a variable representing the first parameter of the command line, this file itself. (`%1`-`%9` represent the parameters passed to the command.)
  - The `~` allows modifiers to the applied to the variable expansion.
  - `d` is the drive letter for where the current file is located.
  - `p` is the path to the directory containing the current file.
  - `ruby` is a literal string added to that drive and path.
* `-x` is a switch to the Ruby interpreter inherited from perl. It means "ignore all the lines above the shebang (`#!`) line in the file you're about to interpret.
* `"%~f0"` is another tweak on "the current file." In this case, it expands to the fully qualified path--drive letter, path to directory containing, and the file's name--of the current file.
* `%*` is "all the parameters `%1` on up from the command line."
So, in English, this line executes the `ruby` runtime that exists in the same directory as this Ruby script, passing this Ruby script in for interpretation, and tells Ruby to ignore (i.e. _not_ interpret) all of these shell shenanigans above the `#!` line.

```
@exit /b %ERRORLEVEL%
```

* `@` our friend, no-echo. Don't show this line to the user when executing.
* `exit` means exit or stop running this script.
* `/b` is an option to `exit` to tell it to stop _only_ this script, not to exit the shell itself.
* `%ERRORLEVEL%` is a variable containing the exit code of the previous command. So, exit this script with the success/failure of the Ruby script run from the previous line.
And thus endeth the processing by the Windows Command Shell. It doesn't try running the command on any of the lines that appear after.

## As UNIXy Shell

Things are much trickier for the sh/bash shells.
{{< highlight shell >}}

:""||{ ""=> %q<-*- ruby -*-
@"%~dp0ruby" -x "%~f0" %*
@exit /b %ERRORLEVEL%
};

{{< / highlight >}}

This chunk sort of needs to be taken as a whole.
* `:` means "true" from the olden days of sh and can work like a no-op operator.
* `||` is a logical OR operator. So what's on the left of it is true.
* `{ }` wraps some characters on the first line that we'll talk about later, plus the Windows Command Shell command into an inline group. So the Command Shell commands are treated as one thing and ignored.
What does just this hunk look like when run through sh/bash? (sh uses `-x` differenly than Ruby and perl. To sh/bash/etc, `-x` means "show each line before it is executed" which is useful for debugging.)

{{< highlight text >}}
$ sh -x test.cmd
+ :
{{< / highlight >}}

It sees nothing! It does nothing!
So, it moves on to the second inline group it sees.

{{< highlight shell >}}

{ #
bindir="${0%/*}" #
exec "$bindir/ruby" -x "$0" "$@" #
>, #
} #

{{< / highlight >}}

* `{ #` starts a new inline group. The `#` on the same line appeases the sh interpreter that really doesn't like for the `{` to be lonesome on a line. This is what the `""=> %q<-*- ruby -*-` characters on the very first line are doing for the opening of the first inline group.

{{< highlight shell >}}

bindir="${0%/*}" #

{{< / highlight >}}

* `bindir=` is setting a local variable named `bindir`
* `"${0%/*}"` is the sh/bash means to determine the path to the directory containing the current file. `$0` is similarly the first command line parameter (this file executing) and `%/*` trims it to only the path of the directory containing this file.
* `#` ... I don't think the comment here matters.

{{< highlight shell >}}
exec "$bindir/ruby" -x "$0" "$@" # -x "$0" "$@" #
{{< / highlight >}}

* `exec` is a shell built-in that replaces the current process being executed with a new one. This is what stops the sh/bash execution of the current script and runs it in Ruby instead.
* `"$bindir/ruby"` is how Ruby gets executed, having determined the location of the current file in the line before. (Remember, these binstubs exist in RubyInstaller in the same directory as the installed `ruby.exe` executable.)
* `-x` again, "Dear Ruby, please ignore all the shenanigans above `#!`."
* `"$0"` is as before but very specifically _this file_ passed in as the input to be interpreted by Ruby.
* `"$@"` is "all the parameters from the command line."
And because this was run with `exec`, thus endeth the sh/bash interpretation of this file.
...
But wait ... what about that `>, #`?
Indeed. It is a "redirect output to `,`." That is not run by sh/bash because of the `exec`, but is present as a parsible thing that is meaningful for the Ruby interpretation of this file.
But wait ... again ... this file is interpreted by Ruby with the `-x` switch so that all of these lines are _ignored_, aren't they?
Yes. But also, no.

## The Ruby Way

Let's bring back the whole thing and interpret it like Ruby.

{{< highlight ruby >}}

:""||{ ""=> %q<-*- ruby -*-
@"%~dp0ruby" -x "%~f0" %*
@exit /b %ERRORLEVEL%
};{ #
bindir="${0%/*}" #
exec "$bindir/ruby" -x "$0" "$@" #
>, #
} #
#!/usr/bin/env ruby
[ ... actual ruby code ... ]

{{< / highlight >}}

Yes, when executed with `-x`, all the lines above `#!` are ignored. But what if, for some reason, someone passes this file in to `ruby` without the `-x`, will it blow up?
No. Because now more of the hieroglyphs have meaning. Let's run some chunks of this through Ruby.

{{< highlight ruby >}}
$ irb
irb(main):001:0> :""
=> :""
{{< / highlight >}}

`:""` is a symbol. Cool. Ruby symbols are true.

{{< highlight ruby >}}
irb(main):002:0> :"" || 'nope'
=> :""
{{< / highlight >}}

So, whatever thing appears to the right of the logical OR (`||`) will be ignored. Let's trim away some things and lay out what's to the right as though it were styled as Ruby code.

{{< highlight ruby >}}

{ "" => %q<-*- ruby -*-
  @"%~dp0ruby" -x "%~f0" %*
  @exit /b %ERRORLEVEL%
  };{ #
  bindir="${0%/*}" #
  exec "$bindir/ruby" -x "$0" "$@" #
>, #
} #

{{< / highlight >}}

That's a hash with one key/value pair. The key is the symbol `:""`. The value is a quoted, multiline string (`%q< >`).

{{< highlight ruby >}}

irb(main):003:0> { "" => %q<-*- ruby -*-
irb(main):004:1' @"%~dp0ruby" -x "%~f0" %*
irb(main):005:1' @exit /b %ERRORLEVEL%
irb(main):006:1' };{ #
irb(main):007:1' bindir="${0%/*}" #
irb(main):008:1' exec "$bindir/ruby" -x "$0" "$@" #
irb(main):009:1' >, #
irb(main):010:1* } #
=> {""=>"-*- ruby -*-\n\n@\"%~dp0ruby\" -x \"%~f0\" %*\n\n@exit /b %ERRORLEVEL%\n\n};{ #\n\nbindir=\"${0%/*}\" #\n\nexec \"$bindir/ruby\" -x \"$0\" \"$@\" #\n\n"}

{{< / highlight >}}

So the hieroglyphs handily wrap all the Windows Command Shell and sh/bash pieces into a string to avoid parsing it as Ruby. And then does nothing with that hash containing the string because it sits on the right side of an `||` with an always true item `:""` on the left. The `#!` line starts with a `#` turning it into a comment. Then on into the actual Ruby portion of the script.
So damn clever. A Ruby script that knows how to run itself through three interpreters.
