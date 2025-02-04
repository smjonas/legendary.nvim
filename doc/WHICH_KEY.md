# `which-key.nvim` Integration

Already a `which-key.nvim` user? Use your existing `which-key.nvim` tables with `legendary.nvim`!

There's a couple ways you can choose to do it:

```lua
-- automatically register which-key.nvim tables with legendary.nvim
-- when you register them with which-key.nvim.
-- `setup()` must be called before `require('which-key).register()`
require('legendary').setup({ which_key = { auto_register = true } })
-- now this will register them with both which-key.nvim and legendary.nvim
require('which-key').register(your_which_key_tables, your_which_key_opts)

-- or, pass them through setup() directly
require('which-key').register(your_which_key_tables, your_which_key_opts)
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
require('legendary.integrations.which-key').bind_whichkey(
  your_which_key_tables,
  your_which_key_opts,
  -- false if which-key.nvim handles binding them,
  -- set to true if you want legendary.nvim to handle binding
  -- the mappings; if not passed, true by default
  false,
)
```
