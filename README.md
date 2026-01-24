# repeatable-move.nvim

A Neovim plugin that allows you to repeat movements to the next or previous instance of any movement functionality.

Since nvim-treesitter-textobjects `main` branch no longer supports repeatable movements with custom functions, this plugin provides a workaround to achieve the same functionality.

> [!NOTE]
> This plugin is NOT an official nvim-treesitter plugin.  
> Since it plugs into nvim-treesitter-textobjects, if they change their internal API, this plugin may break.  
> If that happens, I will do my best to implement a working solution. But for now, this is the simplest way to achieve the desired functionality.

## Installation

Using [lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
{
  "kiyoon/repeatable-move.nvim",
  dependencies = { "nvim-treesitter/nvim-treesitter-textobjects" },
},
```

## Usage

Step 1: follow nvim-treesitter-textobjects for setting up keybinds for `;`, `,`, `f`, `t`, `F`, `T`.

```lua
local ts_repeat_move = require "nvim-treesitter.textobjects.repeatable_move"

-- Repeat movement with ; and ,
-- ensure ; goes forward and , goes backward regardless of the last direction
vim.keymap.set({ "n", "x", "o" }, ";", ts_repeat_move.repeat_last_move_next)
vim.keymap.set({ "n", "x", "o" }, ",", ts_repeat_move.repeat_last_move_previous)

-- vim way: ; goes to the direction you were moving.
-- vim.keymap.set({ "n", "x", "o" }, ";", ts_repeat_move.repeat_last_move)
-- vim.keymap.set({ "n", "x", "o" }, ",", ts_repeat_move.repeat_last_move_opposite)

-- Optionally, make builtin f, F, t, T also repeatable with ; and ,
vim.keymap.set({ "n", "x", "o" }, "f", ts_repeat_move.builtin_f_expr, { expr = true })
vim.keymap.set({ "n", "x", "o" }, "F", ts_repeat_move.builtin_F_expr, { expr = true })
vim.keymap.set({ "n", "x", "o" }, "t", ts_repeat_move.builtin_t_expr, { expr = true })
vim.keymap.set({ "n", "x", "o" }, "T", ts_repeat_move.builtin_T_expr, { expr = true })
```

Step 2: Use this plugin for registering your custom movements.

- Use `make_repeatable_move_pair` function to create a pair of repeatable move functions for next and previous movements.  
- Use `make_repeatable_move` function to create a single repeatable move function.
- Use `set_last_move` function to manually set the last move function if needed.

Example: make [gitsigns.nvim](https://github.com/lewis6991/gitsigns.nvim) movement repeatable with ; and , keys.

```lua
local repeat_move = require("repeatable_move")
local gs = require("gitsigns")

-- make sure forward function comes first
local next_hunk_repeat, prev_hunk_repeat = repeat_move.make_repeatable_move_pair(gs.next_hunk, gs.prev_hunk)

vim.keymap.set({ "n", "x", "o" }, "]h", next_hunk_repeat)
vim.keymap.set({ "n", "x", "o" }, "[h", prev_hunk_repeat)
```

Example: LSP next/previous diagnostic

```lua
vim.api.nvim_create_autocmd("LspAttach", {
  callback = function(args)
    local repeat_move = require("repeatable_move")

    local next_warn = function()
      vim.diagnostic.jump({ count = 1, severity = vim.diagnostic.severity.WARN })
    end
    local prev_warn = function()
      vim.diagnostic.jump({ count = -1, severity = vim.diagnostic.severity.WARN })
    end
    local next_err = function()
      vim.diagnostic.jump({ count = 1, severity = vim.diagnostic.severity.ERROR })
    end
    local prev_err = function()
      vim.diagnostic.jump({ count = -1, severity = vim.diagnostic.severity.ERROR })
    end

    -- make sure forward function comes first
    next_warn, prev_warn = repeat_move.make_repeatable_move_pair(next_warn, prev_warn)
    next_err, prev_err = repeat_move.make_repeatable_move_pair(next_err, prev_err)

    local opts = { buffer = args.buf }
    vim.keymap.set({ "n", "x", "o" }, "]w", next_warn, vim.tbl_extend("force", opts, { desc = "Next warning" }))
    vim.keymap.set({ "n", "x", "o" }, "[w", prev_warn, vim.tbl_extend("force", opts, { desc = "Previous warning" }))
    vim.keymap.set({ "n", "x", "o" }, "]e", next_err, vim.tbl_extend("force", opts, { desc = "Next error" }))
    vim.keymap.set({ "n", "x", "o" }, "[e", prev_err, vim.tbl_extend("force", opts, { desc = "Previous error" }))
  end,
})
```

Example: [todo-comments.nvim](https://github.com/folke/todo-comments.nvim)

```lua
{
  "folke/todo-comments.nvim",
  dependencies = "nvim-lua/plenary.nvim",
  config = function()
    local todo_comments = require("todo-comments")
    local next_todo, prev_todo
    local repeat_move = require("repeatable_move")
    -- make sure forward function comes first
    next_todo, prev_todo = repeat_move.make_repeatable_move_pair(todo_comments.jump_next, todo_comments.jump_prev)
    vim.keymap.set("n", "]t", next_todo, { desc = "Next todo comment" })
    vim.keymap.set("n", "[t", prev_todo, { desc = "Previous todo comment" })
  end,
},
```

Example: [aeriel.nvim](https://github.com/stevearc/aerial.nvim)

```lua
{
  "stevearc/aerial.nvim",
  event = "BufReadPre",
  config = function()
    local aerial = require("aerial")
    aerial.setup({
      on_attach = function(bufnr)
        local anext, aprev
        local repeat_move = require("repeatable_move")
        -- make sure forward function comes first
        anext, aprev = repeat_move.make_repeatable_move_pair(aerial.next, aerial.prev)
        vim.keymap.set("n", "]r", anext, { buffer = bufnr, desc = "Aerial next" })
        vim.keymap.set("n", "[r", aprev, { buffer = bufnr, desc = "Aerial prev" })
      end,
    })
  end,
},
```
