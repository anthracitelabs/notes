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
