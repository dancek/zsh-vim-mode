# zsh-vim-mode

Friendly bindings for ZSH's vi mode

## Installation

Install this plugin with any [ZSH plugin manager][], or just source it from
your `.zshrc`.

[ZSH plugin manager]: https://github.com/unixorn/awesome-zsh-plugins/blob/master/README.md#installation

## Features

### Additional key bindings

In INSERT mode (`viins` keymap), most Emacs key bindings are available. Use
`^A` and `^E` for beginning and end of line, `^R` for incremental search,
etc. `<Esc>` or `<C-[>` quickly switches into NORMAL mode (`vicmd`).

### Surround Bindings for ZSH text objects

ZSH has support for text objects since 5.0.8. This plugin adds the
[suggested bindings][] to use surround-type objects. For example, when in
NORMAL mode with the cursor inside a double-quoted string, type `ci"` to
change the contents of the string. Or type `cs"(` to change the quotes to
parentheses. Type `ds(` to remove the parentheses. Type `ys2W]` to surround
the following two Words with brackets.

In visual mode, type `a[` to select the surrounding bracketed text
(including the brackets), or type `i'` to select the text within single
quotes. Type `S<` to put angle brackets around the selected text.

[suggested bindings]: https://sourceforge.net/p/zsh/code/ci/master/tree/Functions/Zle/surround

### Mode-sensitive cursor styling

Change the color and shape of the terminal cursor with:

```zsh
MODE_CURSOR_VIINS="#00ff00 blinking bar"
MODE_CURSOR_REPLACE="$MODE_CURSOR_VIINS #ff0000"
MODE_CURSOR_VICMD="green block"
MODE_CURSOR_SEARCH="#ff00ff steady underline"
MODE_CURSOR_VISUAL="$MODE_CURSOR_VICMD steady bar"
MODE_CURSOR_VLINE="$MODE_CURSOR_VISUAL #00ffff"
```

Use X11 color names or `#RRGGBB` notation for colors. The recognized
style words are `steady`, `blinking`, `block`, `underline` and `bar`.

If your cursor used to blink, and now it's stopped, you can fix that
with `unset MODE_CURSOR_DEFAULT`. The default (steady) is
appropriate for most terminals.

If you are using `tmux` but `$TMUX` is not set (e.g., you're running
zsh on a remote host), you may need to set `TMUX_PASSTHROUGH=1` to
get the cursor styling to work.

When in VISUAL or VLINE mode, ZSH colors text in reverse (background and
foreground colors swapped). Depending on your terminal, this may override or
interfere with the cursor color. Using `bar` or `underline` may display
better than `block` in some cases.

#### Disabling cursor styling

Cursor styling is not enabled by default. If you do not set any
`MODE_CURSOR_*` variables, the terminal escape sequence to change
the cursor is not sent.

### Mode in prompt

If RPS1 / RPROMPT is not set, the mode indicator will be added
automatically. The appearance can be set with:

```zsh
MODE_INDICATOR_VIINS='%F{15}<%F{8}INSERT<%f'
MODE_INDICATOR_VICMD='%F{10}<%F{2}NORMAL<%f'
MODE_INDICATOR_REPLACE='%F{9}<%F{1}REPLACE<%f'
MODE_INDICATOR_SEARCH='%F{13}<%F{5}SEARCH<%f'
MODE_INDICATOR_VISUAL='%F{12}<%F{4}VISUAL<%f'
MODE_INDICATOR_VLINE='%F{12}<%F{4}V-LINE<%f'
```

If you want to add this to your existing RPS1, there are two ways. If
`setopt prompt_subst` is on, then simply add ${MODE_INDICATOR_PROMPT}
to your RPS1, ensuring it is quoted:

```zsh
setopt PROMPT_SUBST
# Note the single quotes
RPS1='${MODE_INDICATOR_PROMPT} ${vcs_info_msg_0_}'
```

If you do not want to use prompt_subst, then it must **not** be
quoted, and this module must be loaded first before adding it
to your prompt:

```zsh
setopt NO_prompt_subst

# Load this plugin first, then later on ...

MODE_INDICATOR_VICMD='%F{9}<%F{1}<<%f'
MODE_INDICATOR_SEARCH='%F{13}<%F{5}<<%f'
# Note the double quotes
RPS1="${MODE_INDICATOR_PROMPT} %B%F{15}<%b %*"
```

Each time the line editor keymap changes, the *text* of the prompt
will be substituted, removing the previous mode indicator text and
inserting the new.

If your theme sets `$MODE_INDICATOR`, it will be used as a default
for `MODE_INDICATOR_VICMD` if nothing else is set.

#### Disabling mode indicator in prompt

If you set `MODE_INDICATOR=""`  _before_ loading this plugin, and none
of the other `MODE_INDICATOR_*` variables are set, then the prompt
is not modified by this plugin.

### Mode for integration with other plugins

The `$VIM_MODE_KEYMAP` variable is set to `viins`, `vicmd`, `replace`,
`isearch`, `visual` or `vline` for easy inspection from other plugins.

### Changing the initial editing mode

ZSH initially is in INSERT mode (the `viins` keymap) with each new command
prompt. If you want to always start in NORMAL mode (the `vicmd` keymap), set
`VIM_MODE_INITIAL_KEYMAP=vicmd`. If you want to keep the mode you were in on
the last command line, set `VIM_MODE_INITIAL_KEYMAP=last`.

For example, if you type `<Esc>` to switch to NORMAL mode, then type `BBBdw`
to go back three Words and delete a word, you are still in NORMAL mode. If
you type `<Enter>` to submit the command, and `VIM_MODE_INITIAL_KEYMAP` is
set to `last`, you will be placed in NORMAL mode at the next command prompt.

## Compatibility

This plugin uses features added in ZSH 5.3 (`add-zle-hook-widget`, etc.).

### `KEYTIMEOUT`

This plugin sets `KEYTIMEOUT` to a reasonably short duration. If you set it
to `1`, as in many examples on the net, it has been known to cause some
subtle problems. Also note that multi-key commands in NORMAL mode must be
typed within the duration. Eg. if you try to type `cs"(` and the duration
between `c` and `s` is over `KEYTIMEOUT`, the `s` will be treated separately
and will take you back to INSERT mode.

On the other hand, the time it takes for `<esc>` to switch to NORMAL mode 
tends to be `KEYTIMEOUT` as there are bindings beginning with the escape
character and ZLE has to wait to see if the user is typing one of them. A
hard-core workaround is to [remove all bindings starting with `<esc>`](http://zsh.sourceforge.net/Guide/zshguide04.html#l95).
But that includes arrow keys and such. More reasonably, you could unbind
dual escapes; that way you only lose Alt-Left and Alt-Right for word
movement in INSERT mode: 

```
bindkey -rpM viins '^[^['
```

Pressing `<esc><esc>` will then switch to NORMAL mode with no delay.

Please file a report if the current `KEYTIMEOUT` setting is too brief for
you.


## Bugs

If you find this doesn't work with your terminal, your plugins, your
settings or your version of ZSH, please [open an issue][issues]. If
it clobbers some setting that it shouldn't, please open an issue.

It is usually helpful to create a clean `.zshrc` that only contains
`source ~/path-to/zsh-vim-mode/zsh-vim-mode.plugin.zsh`. If your
issue disappears, then please start adding back items from your
configuration until you find one that causes the problem. Put that test
`.zshrc` in the bug report. Thanks!

[issues]: https://github.com/softmoth/zsh-vim-mode/issues

## License

Some of this code is mangled together from blogs, mailing lists, random
repositories, and other plugins. If you have any licensing concerns, please
open an issue so it can be addressed. That being said, to the extent possible:

This code is released under the MIT license.
