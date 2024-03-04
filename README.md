yadi.vim
========

Yet Another Detect Indent. Here is how it works in my `.vimrc`:

```vim
" Try to auto detect and use the indentation of a file when opened. 
autocmd BufRead * DetectIndent

" Otherwise use file type specific indentation. E.g. tabs for Makefiles
" and 4 spaces for Python. This is optional.
filetype plugin indent on

" Set a fallback here in case detection fails and there is no file type
" plugin available. You can also omit this, then Vim defaults to tabs.
set expandtab shiftwidth=4 softtabstop=4

" You stay in control of your tabstop setting.
set tabstop=4
```

The plugin is built on the following principles:

* Ignore outliers. A few bad whitespace should not stop the plugin from
  working. *But* as a user I don't want to be warned of or worse required to fix
  those inconsistencies. That's not in the scope of this plugin.
* Leave settings unchanged if detection fails. The users `.vimrc` or
  [file type plugins](https://vim.fandom.com/wiki/File_type_plugins)
  should then determine the indentation style.
* Only ever touch the
  [3 settings relevant to indentation](https://vim.fandom.com/wiki/Indenting_source_code#Setup):
  `expandtab`, `shiftwidth` and `softtabstop`
* Be a single, short and easy to comprehend Vimscript file.

Not convinced yet?
[Take a look at the code](https://github.com/timakro/vim-yadi/blob/main/plugin/yadi.vim#L17-L62),
it fits on a single page.

Commands
--------

The `:DetectIndent` command tries to auto detect the indentation style in the
current buffer.
If it finds tabs it sets `noexpandtab shiftwidth=0 softtabstop=0`,
if it finds n spaces it sets `expandtab shiftwidth=n softtabstop=n`.
If the algorithm can't confidently determine the indentation style no settings
are changed. It comes naturally to set this as an autocommand as can be seen
above.

The `:IndentTabs` and `:IndentSpaces <n>` commands explicitly apply the
settings for tabs and spaces respectively. The argument to `:IndentSpaces` can
be omitted in which case your tabstop setting will be used.

The algorithm
-------------

The developers of the Firefox dev tools wrote
[a great article](https://medium.com/firefox-developer-tools/detecting-code-indentation-eff3ed0fb56b)
where they compare different indentation detection strategies for Firefox's
built-in source editor. yadi.vim uses the "comparing lines" strategy from the
article because it performs well and is easy to implement. The article explains
it well:

> This method compares the indentation of each line with the previous line, and
> adds the difference to a tally. So if a line is indented by 10 spaces, and
> the previous by 8, one more vote would be added for 2-space indentation.

Mixed tabs and spaces, 1-space indentation, as well as more than 8 spaces are
not supported. This is a deliberate choice as these indentation styles are rare
and would complicate the algorithm considerably.

Furthermore there is a heuristic in place to prevent misdetection. More than
80% of the file must be either tabs or spaces for the detection to succeed.
Additionally in the case of spaces the most common indentation level must make
up more than 60% of all indentations.

Integration with statuslines
----------------------------

To display the current indentation settings in your statusline you could use
this expression:

```vim
let &statusline = '%{&expandtab?shiftwidth()." sp":"tabs"}'
```

Or with [lightline.vim](https://github.com/itchyny/lightline.vim):

```vim
let g:lightline = {
      \ 'active': {
      \   'right': [ [ 'lineinfo' ],
      \              [ 'percent' ],
      \              [ 'fileformat', 'fileencoding', 'filetype', 'indentstyle' ] ]
      \ },
      \ 'component': {
      \   'indentstyle': '%{&expandtab?shiftwidth()." sp":"tabs"}'
      \ },
      \ }
```

Why yet another clone of DetectIndent?
--------------------------------------

There are a bunch of plugins with DetectIndent-like functionality:

* [DetectIndent](https://github.com/ciaranm/detectindent)
* [sleuth.vim](https://github.com/tpope/vim-sleuth)
* [Yaifa](https://github.com/Raimondi/yaifa)
* [matchindent.vim](https://github.com/conormcd/matchindent.vim)
* [IndentFinder](https://github.com/ldx/vim-indentfinder)

But none of them let you fall back to file type plugins if detection fails. Most of them force you to set a fallback indentation style. [sleuth.vim](https://github.com/tpope/vim-sleuth) is more sophisticated and tries to guess the indentation from surrounding files. It's a different approach which comes at the cost of greater complexity and makes the result rather unpredictable.
