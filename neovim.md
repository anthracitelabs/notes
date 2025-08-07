Follow installation instructions on github page.
https://github.com/neovim/neovim

Configuration of Neovim is done through plugins. The first step is to install a plugin manager. lazy.nvim is the recommended plugin manager. Packer was popular before but it is no longer maintained. 
https://lazy.folke.io/

As you can find in the documentation there are two options while installing lazy.nvim; structured vs single file (everything in init.lua file). structured is the recommended approach.

~/.config/nvim is your Neovim configuration directory. On Windows, this is usually ~\AppData\Local\nvim. To know the correct path for your system, run :echo stdpath('config').

Configuration
So for linux inside ~/.config/nvim/init.lua
  require("config.lazy")

Then inside ~/.config/nvim/lua/config/lazy.lua
  -- Bootstrap lazy.nvim
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({ "git", "clone", "--filter=blob:none", "--branch=stable", lazyrepo, lazypath })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- Make sure to setup `mapleader` and `maplocalleader` before
-- loading lazy.nvim so that mappings are correct.
-- This is also a good place to setup other settings (vim.opt)
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Setup lazy.nvim
require("lazy").setup({
  spec = {
    -- import your plugins
    { import = "plugins" },
  },
  -- Configure any other settings here. See the documentation for more details.
  -- colorscheme that will be used when installing plugins.
  install = { colorscheme = { "habamax" } },
  -- automatically check for plugin updates
  checker = { enabled = true },
})



You can then create your plugin specs in ~/.config/nvim/lua/plugins/. Each file should return a table with the plugins you want to install.


~/.config/nvim
├── lua
│   ├── config
│   │   └── lazy.lua
│   └── plugins
│       ├── spec1.lua
│       ├── **
│       └── spec2.lua
└── init.lua

init.lua
-------------
-- disable netrw at the very start of your init.lua
vim.g.loaded_netrw = 1
vim.g.loaded_netrwPlugin = 1

-- optionally enable 24-bit colour
vim.opt.termguicolors = true

vim.cmd("set expandtab")
vim.cmd("set tabstop=2")
vim.cmd("set softtabstop=2")
vim.cmd("set shiftwidth=2")
vim.g.mapleader = " "

vim.api.nvim_set_option("clipboard", "unnamed")

local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"

vim.opt.rtp:prepend(lazypath)

local plugins = {
  { dir = "/home/dev/.config/nvim-data/lazy/tokyonight.nvim", name = "tokyonight", priority = 1000 },

  {
    dir = "/home/dev/.config/nvim-data/lazy/telescope.nvim", tag = '0.1.5', dependencies = { dir = "/home/dev/.config/nvim-data/lazy/plenary.nvim" }
  },

  { dir = "/home/dev/.config/nvim-data/lazy/nvim-treesitter", build = ":TSUpdate" },

  { dir = "/home/dev/.config/nvim-data/lazy/nvim-tree.lua" },

  { dir = "/home/dev/.config/nvim-data/lazy/telescope-fzf-native.nvim", build = 'cmake -S. -Bbuild -DCMAKE_BUILD_TYPE=Release && cmake --build build --config Release' }

}

local opts = {}

require("lazy").setup(plugins, opts)

local builtin = require("telescope.builtin")
vim.keymap.set('n', '<leader>ff', builtin.find_files, { desc = 'Telescope find files' })
vim.keymap.set('n', '<leader>fg', builtin.live_grep, { desc = 'Telescope live grep' })
vim.keymap.set('n', '<leader>fb', builtin.buffers, { desc = 'Telescope buffers' })
vim.keymap.set('n', '<leader>fh', builtin.help_tags, { desc = 'Telescope help tags' })

local telescope = require("telescope")
telescope.setup({
	pickers = {
		live_grep = {
			file_ignore_patterns = { 'node_modules', '.git', '.venv' },
			additional_args = function(_)
				return {"--hidden"}
			end
		},
	
		find_files = {
			file_ignore_patterns = { '.git', '.venv' },
			hidden = true
		}
	},
})

-- empty setup using defaults
require("nvim-tree").setup()

local api = require("nvim-tree.api")
vim.keymap.set('n', '<C-g>', api.tree.focus, {})
vim.keymap.set('n', '<C-h>', api.tree.find_file, {})

-- set color scheme to tokyonight at start
vim.cmd[[colorscheme tokyonight]]

vim.keymap.set('n', '<space>st', function()
	vim.cmd.vnew()
	vim.cmd.term()
  vim.cmd.wincmd("J")
	vim.api.nvim_win_set_height(0, 10)
end)



