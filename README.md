# Legendary

Define your keymaps, commands, and autocommands as simple Lua tables, building a legend at the same time.

<!-- panvimdoc-ignore-start -->

![demo](https://user-images.githubusercontent.com/8648891/160112850-5fbbf327-309c-4bac-ad6b-df217127d886.gif)
<sup>Theme used in recording is [lighthaus.nvim](https://github.com/mrjones2014/lighthaus.nvim). The finder UI is handled by [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) via [dressing.nvim](https://github.com/stevearc/dressing.nvim). See [Prerequisites](#prerequisites) for details.</sup>

<!-- panvimdoc-ignore-end -->

## Features

- Define your keymaps, commands, and `augroup`/`autocmd`s as simple Lua tables, then bind them with `legendary.nvim`
- Integration with [which-key.nvim](https://github.com/folke/which-key.nvim), use your existing `which-key.nvim` tables with `legendary.nvim`
- Anonymous mappings -- show mappings/commands in the finder without having `legendary.nvim` handle creating them
- Uses `vim.ui.select()` so it can be hooked up to a fuzzy finder using something like [dressing.nvim](https://github.com/stevearc/dressing.nvim) for a VS Code command palette like interface
- Execute normal, insert, and visual mode keymaps, commands, and autocommands, when you select them
- Show your most recently executed keymap, command, function or autocmd at the top when triggered via `legendary.nvim` (can be disabled via config)
- Buffer-local keymaps, commands, functions and autocmds only appear in the finder for the current buffer
- Help execute commands that take arguments by prefilling the command line instead of executing immediately
- Search built-in keymaps and commands along with your user-defined keymaps and commands (may be disabled in config). Notice some missing? Comment on [this discussion](https://github.com/mrjones2014/legendary.nvim/discussions/89) or submit a PR!
- A `legendary.helpers` module to help create lazily-evaluated keymaps and commands. Have an idea for a new helper? Comment on [this discussion](https://github.com/mrjones2014/legendary.nvim/discussions/90) or submit a PR!

## Prerequisites

- Neovim 0.7.0+; specifically, this plugin depends on the following APIs:
  - `vim.keymap.set`
  - `vim.api.nvim_create_augroup`
  - `vim.api.nvim_create_autocmd`
- (Optional) A `vim.ui.select()` handler; this provides the UI for the finder.
  - I recommend [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) paired with [dressing.nvim](https://github.com/stevearc/dressing.nvim).

## Installation

With `packer.nvim`:

```lua
use({'mrjones2014/legendary.nvim'})
```

With `vim-plug`:

```VimL
Plug "mrjones2014/legendary.nvim"
```

## Usage

To trigger the finder for your configured keymaps, commands, and `augroup`/`autocmd`s:

Commands:

```VimL
" search keymaps, commands, and autocmds
:Legendary

" search keymaps
:Legendary keymaps

" search commands
:Legendary commands

" search functions
:Legendary functions

" search autocmds
:Legendary autocmds
```

Lua API:

The `require('legend').find()` function takes an `opts` table with the following fields (all optional):

```lua
{
  -- pass 'keymaps', 'commands', or 'autocmds' to search only one type of item
  kind = nil,
  -- pass a list of filter functions or a single filter function with
  -- the signature `function(item): boolean`
  -- `require('legendary.filters').mode(mode)` and `require('legendary.filters').current_mode()`
  -- are provided for convenience
  filters = {},
  -- pass a function with the signature `function(item, mode): {string}`
  -- returning a list of strings where each string is one column
  -- use this to override the configured formatter for just one call
  formatter = nil,
  -- pass a string, or a function with the signature `function(kind: string): string`
  -- to customize the select prompt for the current call
  select_prompt = nil,
}
```

Examples:

```lua
-- filter keymaps by current mode
require('legendary').find({ filters = require('legendary.filters').current_mode() })
-- filter keymaps by normal mode
require('legendary').find({ filters = require('legendary.filters').mode('n') })
-- show only keymaps and filter by normal mode
require('legendary').find({ kind = 'keymaps', filters = require('legendary.filters').mode('n') })
-- customize the select prompt
require('legendary').find({ select_prompt = 'Custom prompt' })
require('legendary').find({
  kind = 'keymaps',
  select_prompt = function(kind)
    return string.format('Finding %s', kind)
  end
})
-- filter keymaps by normal mode and that start with <leader>
require('legendary').find({
  filters = {
    require('legendary.filters').mode('n'),
    function(item)
      if not string.find(item.kind, 'keymap') then
        return true
      end

      return vim.startswith(item[1], '<leader>')
    end
  }
})
-- filter keymaps by current mode, and only display current mode in first column
require('legendary').find({
  filters = { require('legendary.filters').current_mode() },
  formatter = function(item, mode)
    local values = require('legendary.formatter').get_default_format_values(item)
    if string.find(item.kind, 'keymap') then
      values[1] = mode
    end
    return values
  end
})
```

## Configuration

Default configuration is shown below. For a detailed explanation of the structure for
keymap, command, and `augroup`/`autocmd` tables, see [Table Structures](#table-structures).

```lua
require('legendary').setup({
  -- Include builtins by default, set to false to disable
  include_builtin = true,
  -- Include the commands that legendary.nvim creates itself
  -- in the legend by default, set to false to disable
  include_legendary_cmds = true,
  -- Customize the prompt that appears on your vim.ui.select() handler
  -- Can be a string or a function that takes the `kind` and returns
  -- a string. See "Item Kinds" below for details. By default,
  -- prompt is 'Legendary' when searching all items,
  -- 'Legendary Keymaps' when searching keymaps,
  -- 'Legendary Commands' when searching commands,
  -- and 'Legendary Autocmds' when searching autocmds.
  select_prompt = nil,
  -- Optionally pass a custom formatter function. This function
  -- receives the item as a parameter and the mode that legendary
  -- was triggered from (e.g. `function(item, mode): {string}`)
  -- and must return a table of non-nil string values for display.
  -- It must return the same number of values for each item to work correctly.
  -- The values will be used as column values when formatted.
  -- See function `get_default_format_values(item)` in
  -- `lua/legendary/formatter.lua` to see default implementation.
  formatter = nil,
  -- When you trigger an item via legendary.nvim,
  -- show it at the top next time you use legendary.nvim
  most_recent_item_at_top = true,
  -- Initial keymaps to bind
  keymaps = {
    -- your keymap tables here
  },
  -- Initial commands to bind
  commands = {
    -- your command tables here
  },
  -- Initial functions to bind
  functions = {
    -- your function tables here
  },
  -- Initial augroups and autocmds to bind
  autocmds = {
    -- your autocmd tables here
  },
  which_key = {
    -- you can put which-key.nvim tables here,
    -- or alternatively have them auto-register,
    -- see section on which-key integration
    mappings = {},
    opts = {},
    -- controls whether legendary.nvim actually binds they keymaps,
    -- or if you want to let which-key.nvim handle the bindings.
    -- if not passed, true by default
    do_binding = {},
  },
  -- Automatically add which-key tables to legendary
  -- see "which-key.nvim Integration" below for more details
  auto_register_which_key = false,
  -- settings for the :LegendaryScratch command
  scratchpad = {
    -- configure how to show results of evaluated Lua code,
    -- either 'print' or 'float'
    -- Pressing q or <ESC> will close the float
    display_results = 'float',
    -- cache the contents of the scratchpad to a file and restore it
    -- next time you open the scratchpad
    cache_file = string.format('%s/%s', vim.fn.stdpath('cache'), 'legendary_scratch.lua'),
  },
})
```

### `which-key.nvim` Integration

Already a `which-key.nvim` user? Use your existing `which-key.nvim` tables with `legendary.nvim`!

There's a couple ways you can choose to do it:

```lua
-- automatically register which-key.nvim tables with legendary.nvim
-- when you register them with which-key.nvim.
-- `setup()` must be called before `require('which-key).register()`
require('legendary').setup({ auto_register_which_key = true })
-- now this will register them with both which-key.nvim and legendary.nvim
require('which-key').register(your_which_key_tables, your_which_key_opts)

-- or, pass them through setup() directly
require('legendary').setup({
  which_key = {
    mappings = your_which_key_tables,
    opts = your_which_key_opts,
    -- false if which-key.nvim handles binding them,
    -- set to true if you want legendary.nvim to handle binding
    -- the mappings; if not passed, true by default
    do_binding = false,
  },
})

-- or, if you'd prefer to manually register with legendary.nvim
require('which-key').register(your_which_key_tables, your_which_key_opts)
require('legendary').bind_whichkey(
  your_which_key_tables,
  your_which_key_opts,
  -- false if which-key.nvim handles binding them,
  -- set to true if you want legendary.nvim to handle binding
  -- the mappings; if not passed, true by default
  false,
)
```

## Table Structures

The tables for keymaps, commands, and `augroup`/`autocmd`s are all similar.

Descriptions can be specified either in the top-level `description` property
on each table, or inside the `opts` table as `opts.desc = 'Description goes here'`.

For `autocmd`s, you must include a `description` property for it to appear in the finder.
This is a design decision because keymaps and commands are frequently executed manually,
so they should appear in the finder by default, while executing `autocmd`s manually with
`:doautocmd` is a much less common use-case, so `autocmd`s are hidden from the finder
unless a description is provided.

You can also run `:LegendaryApi`, then search for `/legendary.types` to read the full API
documentation on accepted types.

<!-- panvimdoc-ignore-start -->
<details>
<summary>
<h3>
<!-- panvimdoc-ignore-end -->

Keymaps

<!-- panvimdoc-ignore-start -->
</h3>
</summary>
<!-- panvimdoc-ignore-end -->

For keymaps you are mapping yourself (as opposed to mappings set by other plugins),
the first two elements are the key and the handler, respectively. The handler
can be a command string like `:wa<CR>` or a Lua function. Example:

```lua
local keymaps = {
  { '<leader>s', ':wa<CR>', description = 'Write all buffers', opts = {} },
  { '<leader>fm', vim.lsp.buf.formatting_sync, description = 'Format buffer with LSP' },
}
```

For "anonymous" mappings (you want them to appear in the finder but don't want `legendary.nvim`
to handle creating them, e.g. for plugin keymappings), just omit the second entry (the "implementation"):

```lua
local keymaps = {
  { '<leader>s', description = 'Write all buffers', opts = {} },
  { '<leader>fm', description = 'Format buffer with LSP' },
}
```

If you need to pass parameters to the Lua function or call a function dynamically from a plugin,
you can use the following helper functions:

```lua
local helpers = require('legendary.helpers')
local keymaps = {
  { '<leader>p', helpers.lazy(vim.lsp.buf.formatting_sync, nil, 1500), description = 'Format with 1.5s timeout' },
  { '<leader>f', helpers.lazy_required_fn('telescope.builtin', 'oldfiles', { only_cwd = true }) },
  -- you can use a dot in the 2nd parameter to access functions nested in tables
  { '<leader>tt', helpers.lazy_required_fn('neotest', 'run.run') },
}
```

The keymap's mode defaults to normal (`n`), but you can set a different mode, or list of modes, via
the `mode` property:

```lua
local keymaps = {
  { '<leader>c', ':CommentToggle<CR>', description = 'Toggle comment', mode = { 'n', 'v' } }
}
```

Alternatively, you can map separate implementations for each mode by passing the second
element as a table, where the table keys are the modes:

```lua
local keymaps = {
  { '<leader>c', { n = ':CommentToggle<CR>', v = ':VisualCommentToggle<CR>' }, description = 'Toggle comment' }
}
```

If you need to pass separate opts per-mode, you can do that too:

```lua
local keymaps = {
  {
    '<leader>c',
    {
      n = { ':CommentToggle<CR>' opts = { noremap = true } },
      v = { ':VisualCommentToggle<CR>' opts = { silent = false } }
    },
    description = 'Toggle comment'
    -- if outer opts exist, the inner opts tables will be merged,
    -- with the inner opts taking precedence
    opts = { expr = false }
  }
}
```

If you want the per-mode mappings to be treated as separate keymaps,
you can specify a separate description per-mode:

```lua
local keymaps = {
  {
    '<leader>c',
    {
      n = {
        ':Something<CR>',
        description = 'Something in normal mode',
        opts = { noremap = true }
      },
      v = {
        ':SomethingElse<CR>'
        opts = {
          -- you can also specify description through opts.desc
          -- if you prefer
          desc = 'Something else in visual mode',
          silent = false,
        }
      }
    },
    description = 'Toggle comment'
    -- if outer opts exist, the inner opts tables will be merged,
    -- with the inner opts taking precedence
    opts = { expr = false }
  }
}
```

You can also pass options to the keymap via the `opts` property, see `:h vim.keymap.set` to
see available options.

```lua
local keymaps = {
  {
    '<leader>fm',
    vim.lsp.buf.formatting_sync,
    description = 'Format buffer with LSP',
    opts = { silent = true, noremap = true }
  },
}
```

If you want a keymap to apply to both normal and insert mode, use a Lua function.

If called in visual mode, the function will be given a table containing the visual
selection range (the marks will also be set) of the form `{ cline, ccol, vline, vcol }`,
where `c` corresponds to the cursor position, and `v` the visual selection (see `:h line()` and `:h col()`). This allows you to create mappings like:

```lua
local keymaps = {
  {
    '<leader>c',
    function(visual_selection)
      if visual_selection then
        -- comment a visual block
        vim.cmd(":'<,'>CommentToggle")
      else
        -- comment a single line from normal mode
        vim.cmd(':CommentToggle')
      end
    end,
    description = 'Toggle comment',
    mode = { 'n', 'v' },
  }
}
```

Finally, if you want to register keymaps with `legendary.nvim` in order to see them in the finder, but not bind
them (like for keymaps set by other plugins), you can just omit the handler element:

```lua
local keymaps = {
  { '<C-d>', description = 'Scroll docs up' },
  { '<C-f>', description = 'Scroll docs down' },
}
```

<!-- panvimdoc-ignore-start -->
</details>
<!-- panvimdoc-ignore-end -->

<!-- panvimdoc-ignore-start -->
<details>
<summary>
<h3>
<!-- panvimdoc-ignore-end -->

Commands

<!-- panvimdoc-ignore-start -->
</h3>
</summary>
<!-- panvimdoc-ignore-end -->

Command tables follow the exact same structure as keymaps, but specify a command name instead of a key code.

```lua
local commands = {
  { ':DoSomething', ':echo "something"', description = 'Do something!' },
  { ':DoSomethingWithLua', require('some-module').some_method, description = 'Do something with Lua!' },
  -- a command from a plugin, don't specify a handler
  { ':CommentToggle', description = 'Toggle comment' },
}
```

You can also pass options to the command via the `opts` property, see `:h nvim_create_user_command` to
see available options. In addition to those options, `legendary.nvim` adds handling for an additional
`buffer` option (a buffer handle, or `0` for current buffer), which will cause the command to be bound
as a buffer-local command.

If you need a command to take an argument, specify `unfinished = true` to pre-fill the command line instead
of executing the command on selected. You can put an argument name/hint in `[]` or `{}` that will be stripped
when filling the command line.

```lua
local commands = {
  { ':MyCommand {some_argument}<CR>', description = 'Command with argument', unfinished = true },
  -- or
  { ':MyCommand [some_argument]<CR>', description = 'Command with argument', un
```
