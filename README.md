# nvim-config

My Neovim config, based on [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim).

- Plugins are managed by the built-in `vim.pack`; versions are pinned in `nvim-pack-lock.json`.
- LSP: `clangd`, `pyright`, `lua_ls` (installed through Mason).
- C/C++ debugger: nvim-dap + nvim-dap-ui with `codelldb` (installed through Mason). See [Debugging C/C++](#debugging-cc).
- Fuzzy finding with Telescope, file tree with Neo-tree (`\`), Treesitter highlighting.
- Leader key is `<space>`.

Requires **Neovim 0.12 or newer**. Distro packages (e.g. apt) are usually too old.

## Install on Linux (Debian/Ubuntu)

```bash
# 1. Dependencies
sudo apt update
sudo apt install -y git make gcc unzip curl ripgrep fd-find xclip
mkdir -p ~/.local/bin
ln -sf "$(which fdfind)" ~/.local/bin/fd      # Ubuntu names the binary fdfind

# 2. Neovim 0.12+ (x86_64; for ARM use the nvim-linux-arm64 tarball)
curl -LO https://github.com/neovim/neovim/releases/download/v0.12.6/nvim-linux-x86_64.tar.gz
tar xzf nvim-linux-x86_64.tar.gz -C ~/.local
ln -sf ~/.local/nvim-linux-x86_64/bin/nvim ~/.local/bin/nvim

# 3. tree-sitter CLI (Treesitter builds parsers with it)
curl -L https://github.com/tree-sitter/tree-sitter/releases/latest/download/tree-sitter-linux-x64.gz | gunzip > ~/.local/bin/tree-sitter
chmod +x ~/.local/bin/tree-sitter

# 4. Config
git clone git@github.com:TruongGiangPham-1/nvim-config.git ~/.config/nvim
nvim
```

Make sure `~/.local/bin` is on your `PATH` (add `export PATH="$HOME/.local/bin:$PATH"` to `~/.bashrc`).

On first launch the plugins install and Mason downloads the language servers and the `codelldb` debugger. Wait for it to finish, then restart nvim. Run `:checkhealth` if anything looks off.

Other distros: install the same tools with your package manager (`ripgrep`, `fd`, `git`, `make`, `gcc`, `unzip`, and `xclip` or `wl-clipboard`).

## Install on WSL2

Follow the Linux steps above, plus the following.

**Line endings (important).** If git has `core.autocrlf=true` (the Windows default), plugins are cloned with CRLF line endings and Vim script files fail with `E488: Trailing characters: ^M`. Fix it before the first launch:

```bash
git config --global core.autocrlf input
```

If plugins were already cloned with CRLF, delete `~/.local/share/nvim` and start nvim again.

**Clipboard.** The config sets `clipboard=unnamedplus`, so yank and paste use the Windows clipboard. Neovim needs a provider for that:

- Windows 11 with WSLg: `xclip` or `wl-clipboard` (installed above) works.
- Otherwise use `win32yank.exe`:
  ```bash
  curl -sLo /tmp/win32yank.zip https://github.com/equalsraf/win32yank/releases/latest/download/win32yank-x64.zip
  unzip -p /tmp/win32yank.zip win32yank.exe > ~/.local/bin/win32yank.exe
  chmod +x ~/.local/bin/win32yank.exe
  ```

Verify with `:checkhealth provider`.

**Icons.** `vim.g.have_nerd_font` is `false` in `init.lua`. If you install a Nerd Font in Windows Terminal, set it to `true`.

## Keys

| Key | Action |
| --- | --- |
| `<space>sf` | Search files |
| `<space>sg` | Grep project |
| `<space><space>` | Open buffers |
| `\` | Toggle file tree |
| `<C-h>` / `<C-l>` | Move between windows |
| `grd` | Go to definition (with LSP attached) |

## Debugging C/C++

Breakpoints and stepping work like the VS Code debugger, using [nvim-dap](https://github.com/mfussenegger/nvim-dap), [nvim-dap-ui](https://github.com/rcarriga/nvim-dap-ui) and `codelldb`. The setup is in `lua/kickstart/plugins/debug.lua`.

1. Compile with debug info and no optimization, otherwise you can't step through the source:
   ```bash
   g++ -g -O0 main.cpp -o main
   ```
2. In nvim, put the cursor on a line and press `<space>b` to set a breakpoint.
3. Press `F5` and pick **LLDB: Launch** (or **LLDB: Launch (args)** to pass arguments). Enter the path to the executable, e.g. `./main`. Relative paths are resolved from the directory nvim was started in.
4. The debug panels (variables, watches, call stack, breakpoints) open automatically and close when the program exits.

| Key | Action |
| --- | --- |
| `F5` | Start / continue |
| `F2` | Step over |
| `F1` | Step into |
| `F3` | Step out |
| `<space>b` | Toggle breakpoint |
| `<space>B` | Conditional breakpoint |
| `F7` | Toggle the debug panels (see the last session's output) |

If `F5` says there is no adapter, check that `codelldb` is installed with `:Mason`.

The system `gdb` is not used. GDB only supports DAP from version 14, and Ubuntu 22.04 ships 12.

## Updating

`:lua vim.pack.update()` updates plugins. Commit `nvim-pack-lock.json` afterwards so other machines get the same versions.
