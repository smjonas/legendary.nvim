# Built-in Filters

The `require('legendary.filters')` module provides the following filters
for convenience:

```lua
local f = require('legendary.filters')

-- take two or more other filter functions
-- and logical AND them
f.AND(...)

-- example
-- show only normal mode keymaps
f.AND(f.mode('n'), f.keymaps())

-- take two or more other filter functions
-- and logical OR them
f.OR(...)

-- example
-- show only keymaps and commands
f.OR(f.keymaps(), f.commands())

-- filter keymaps to ones for a specific mode,
-- other item types are unfiltered since they are
-- not tied to a mode
f.mode(desired_mode)

-- shortcut for vim.api.nvim_get_mode().mode or 'n'
f.current_mode()

-- show only keymaps (and item groups)
f.keymaps()

-- show only keymaps (and item groups)
f.commands()

-- show only autocmds (and item groups)
f.autocmds()

-- show only functions (and item groups)
f.funcs()
```
