---
type: post
title: "Trailing Whitespace Must Die"
date: 2017-06-27 13:19:05 -0400
published: false
comments: true
tags:
---

Or "How I Learned to Stop Worrying And Get My Editor To Do This"

![WHITESPAAAAACE](http://i.imgur.com/JeYcDUQ.jpg)

The following shall be the preferences developers add to their editors for sane handling of whitespace. Sane handling requires:
* default indent is set to 2 spaces
* translating TABs to spaces
* highlighting trailing whitespace from the end of lines
* ensuring that any plain-text file ends with a blank newline

### GO :(

The Go community has decided to enforce code formatting behaviors.  As such they have provided a tool which will change your source code automatically (with or without your permission).  The following are options that "we" do not share with Go, but that will be forced upon you.  Please make every effort to stick to Go's standards when dealing with go.

1. default indent is 8 spaces
1. default indent character is TAB

## XCode

Xcode > Preferences > Text Editing > While editing:

Check "Automatically trim trailing whitespace" and "including white-space only lines".

## Sublime Text 2

Open your personal preferences (⌘-,) and add this:

{{< highlight ruby >}}
"tab_size": 2,
"translate_tabs_to_spaces": true,
"draw_white_space": true,
"ensure_newline_at_eof_on_save": true
{{< / highlight >}}
Another option is the [TrailingSpaces](https://github.com/SublimeText/TrailingSpaces) plugin

### Wild Side

If you want to automatically trim trailing whitespace from lines:

{{< highlight ruby >}}
"trim_trailing_white_space_on_save": true
{{< / highlight >}}

## VIM

Vi already adds a newline to files that need it, so these settings are about dealing with tabs and spaces. Add these lines to your `~/.vimrc`.

{{< highlight vim >}}
" Highlight trailing whitespace in red, http://vim.wikia.com/wiki/Highlight_unwanted_spaces
highlight ExtraWhitespace ctermbg=darkred guibg=#382424
autocmd ColorScheme * highlight ExtraWhitespace ctermbg=red guibg=red
autocmd BufWinEnter * match ExtraWhitespace /\s\+$/
" the above flashes annoyingly while typing, be calmer in insert mode
autocmd InsertLeave * match ExtraWhitespace /\s\+$/
autocmd InsertEnter * match ExtraWhitespace /\s\+\%#\@<!$/

augroup myfiletypes
  "clear old autocmds in group
  autocmd!
  "for ruby, autoindent with two spaces, always expand tabs
  "sw=shiftwidth sts=softtabstop et=expandtabs
  autocmd FileType ruby,haml,eruby,yaml set sw=2 sts=2 et
  autocmd FileType python set sw=4 sts=4 et
  autocmd FileType javascript set sw=2 sts=2 et
augroup END
{{< / highlight >}}

### Adding a command for removing trailing whitespace (~/.vimrc)

{{< highlight vim >}}
  " Remove trailing whitespace from the entire file
  command Rws %s/\s\+$
  map <F5> :Rws
{{< / highlight >}}

Now you can run the command in vim with `F5` or `:Rws`

Also, if you are not yet an expert at tuning your custom VIM config, consider
[Janus](https://github.com/carlhuda/janus) for more options.

## TextMate

Install the [Avian Missing](https://github.com/elia/avian-missing.tmbundle#strip-trailing-whitespace-on-save) bundle.

### Wild Side

Automatically strip trailing white space with the plugin by adding the following to `.tm_properties`:

{{< highlight ruby >}}
TM_STRIP_WHITESPACE_ON_SAVE = true
{{< / highlight >}}

## Emacs

If you use Emacs, you're probably all about [knowing your options](http://www.emacswiki.org/emacs/DeletingWhitespace).

### Wild Side

{{< highlight text >}}
(add-hook 'before-save-hook 'delete-trailing-whitespace)
{{< / highlight >}}
