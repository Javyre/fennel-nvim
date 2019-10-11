# fennel-nvim [WIP]

**Experimental** plugin adding native [fennel](https://fennel-lang.org) support to nvim by utilizing neovim's native lua support. No RPC necessary!

I will likely extend this in the future, but for now I'm just testing out the idea. It should be possible reach seamless integration with a little extra work.

Currently bundled with Fennel `0.4.0-dev`, from the current master branch as of
[this commit](https://github.com/bakpakin/Fennel/tree/d3620b369a6d16f55f98d939a3b47b56d97b9f19).

This is basically just `0.3.0` with an extra docstring or two and better readline completion from the repl
(which requires `readline.lua` to be installed and present on package.path).
In the future, I will switch this over to only bundling a full release, and allowing an option to easily supply a custom version.

- [Install](#install)
- [Fennel Environment](#fennel-environment)
- [Usage](#usage)
  - [init.fnl](#initfnl)
  - [Commands](#commands)
- [Configuration + Lua/Fennel API](#configuration-and-usage-from-lua)
  - [Automatic package.path --> fennel.path sync](#automatic-packagepath----fennelpath-sync)

## Install

The usual, either copy to your nvim config dir or use vim-plug or your plugin manager of choice e.g.

```viml
Plug 'jaawerth/fennel-nvim'
```

## Fennel Environment

This plugin runs your Fennel code under a custom environment which you can alter by passing, as a second argument
to the functions below, an `env` table. It inherits from nvim's `_G`, so you will have access to all normal globals,
including neovim's Lua API.

In addition to the above, for convenient it exposes the `fennelview` function/module as `view` for easy access.


## Usage

The following allows you to run fennel code via Lua in neovim.
For the Lua API to manipulate neovim from Lua/Fennel, see `:help lua-vim`, `:help lua`, and `:help api`.

### `init.fnl`

This plugin, when installed on your `runtimepath`, will automatically look for an `init.fnl` file in your
neovim configuration directories (see `:help xdg` and `:help stdpath`). On Linux, this defaults to `$HOME/.config/nvim/`.

You can disable this behavior as follows in your `init.vim`:

```viml
let g:fennel_nvim_auto_init = v:false
```

The setting can also be read and changed from Lua:

```lua
require('fennel-nvim').autoInit(false) -- disable
require('fennel-nvim').autoInit(true) -- enable
require('fennel-nvim').autoInit() -- get value
```

### Commands

Using the `:Fnl` command:

```viml
" via Fnl command
:Fnl (doc doc)
" Output:
" (doc x)
"   Print the docstring and arglist for a function, macro, or special form.
```

**Note:** Unlike `:lua`, will not work with heredoc (`<<`) syntax, as that is only available to built-in
commands. This behavior may become available in the future when neovim implements `:here`
(per [this issue](https://github.com/neovim/neovim/issues/7638)).

Using the `fnl#eval(code[, arg][, compileOpts])` function - like `luaeval()`, you can pass an argument,
which will be bound to the magic global `_A` in the running environment.

```viml
:call fnl#eval('(print (.. "Hello, " _A "!"))', 'World')
" outputs: Hello, World!
```

### Run a file
With `:FnlFile path/to/file.fnl`

For example, if editing some fennel code you want to test in neovim itself,
```viml
:FnlFile %
```

Similarly, you can use `fnl#eval(filepath[, compileOpts])`.

## Configuration and usage from Lua

**TODO:** Further document this API
The [fennel-nvim](lua/fennel-nvim.lua) Lua module offers an API you can use to eval/load/compile Lua.

```lua
local fnl = require('fennel-nvim')

fnl.dofile('path/to/file.fnl')

-- compile some Fennel into Lua for writing
local compiledLua = fnl.compile('path/to/file.fnl')
```

### Automatic package.path --> fennel.path sync

Because neovim sets `package.path` dynamically on the Lua side based on changes to the `runtimepath`
setting, I've implemented some code that syncs these changes over to `fennel.path`, replacing `?.lua`
with `?.fnl`, `/lua/?/init.lua` with `/fnl/?/init.fnl`, etc.

This behavior can be disabled before running any of the Fennel-executing Vim commands as follows:

```lua
local fnlNvim = require('fennel-nvim')
fnlNvim.syncFennelPath = false -- disabling syncing
fnlNvim.resetFennelPath() -- restore to state before sync
```

