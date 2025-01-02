# 🐽 Bacon Language Server 🐽

[![Ci](https://img.shields.io/github/actions/workflow/status/crisidev/bacon-ls/ci.yml?style=for-the-badge)](https://github.com/crisidev/bacon-ls/actions?query=workflow%3Aci)
[![Release](https://img.shields.io/github/actions/workflow/status/crisidev/bacon-ls/release.yml?style=for-the-badge)](https://github.com/crisidev/bacon-ls/actions?query=workflow%3Arelease)
[![Crates.io](https://img.shields.io/crates/v/bacon-ls?style=for-the-badge)](https://crates.io/crates/bacon-ls)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](https://github.com/crisidev/bacon-ls/blob/main/LICENSE)

**NOTE: bacon-ls 0.5 has breaking changes and will work only with bacon 3.7+. The README for bacon-ls 0.4
can be found [here](./README-0.4.md).**

LSP Server wrapper for the exceptional [Bacon](https://dystroy.org/bacon/) exposing [textDocument/diagnostic](https://microsoft.github.io/language-server-protocol/specification#textDocument_diagnostic) and [workspace/diagnostic](https://microsoft.github.io/language-server-protocol/specification#workspace_diagnostic) capabilities.

See `bacon-ls` 🐽 blog post: https://lmno.lol/crisidev/bacon-language-server

![Bacon screenshot](./img/screenshot.png)

`bacon-ls` 🐽 is meant to be easy to include in your IDE configuration.



* [Roadmap to 1.0 - ✅ done 🕖 in progress 🌍 future](#roadmap-to-1.0---✅-done-🕖-in-progress-🌍-future)
* [Installation](#installation)
    * [VSCode](#vscode)
    * [Mason.nvim](#mason.nvim)
    * [Manual](#manual)
* [Configuration](#configuration)
    * [Neovim - LazyVim](#neovim---lazyvim)
    * [Neovim - Manual](#neovim---manual)
    * [VSCode](#vscode)
* [Troubleshooting](#troubleshooting)
    * [Neovim](#neovim)
    * [VSCode](#vscode)
* [How does it work?](#how-does-it-work?)
* [Thanks](#thanks)

<!-- vim-markdown-toc -->
## Roadmap to 1.0 - ✅ done 🕖 in progress 🌍 future

- ✅ Implement LSP server interface for `textDocument/diagnostic` and `workspace/diagnostic`
- ✅ Manual Neovim configuration
- ✅ Manual [LazyVim](https://www.lazyvim.org) configuration
- 🕖 Automatic NeoVim configuration
  - ✅ Add `bacon-ls` to [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig/) - https://github.com/neovim/nvim-lspconfig/pull/3160
  - ✅ Add `bacon` and `bacon-ls` to [mason.nvim](https://github.com/williamboman/mason.nvim) - https://github.com/mason-org/mason-registry/pull/5774
  - ✅ Add `bacon-ls` to LazyVim [Rust extras](https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/plugins/extras/lang/rust.lua) - https://github.com/LazyVim/LazyVim/pull/3212
- ✅ Add compiler hints to [Bacon](https://dystroy.org/bacon/) export locations - https://github.com/Canop/bacon/pull/187 https://github.com/Canop/bacon/pull/188
- ✅ Support correct span in [Bacon](https://dystroy.org/bacon/) export locations - working from Bacon 3.7
- ✅ VSCode extension and configuration - available on the [release](https://github.com/crisidev/bacon-ls/releases) page from v0.6.0
- 🌍 VSCode extension published available on Marketplace
- 🌍 Emacs configuration

![Bacon gif](./img/bacon-ls.gif)

## Installation

### VSCode

Until the extension is published on the VSCode Marketplace, please download it from the [release](https://github.com/crisidev/bacon-ls/releases) 
and install it manually.

### Mason.nvim

Both Bacon and Bacon-ls are installable via [mason.nvim](https://github.com/williamboman/mason.nvim):

```bash
:MasonInstall bacon bacon-ls
```

### Manual

First, install [Bacon](https://dystroy.org/bacon/#installation) and `bacon-ls` 🐽

```bash
❯❯❯ cargo install --locked bacon bacon-ls
❯❯❯ bacon --version
bacon 3.7.0  # make sure you have at least 3.7.0
❯❯❯ bacon-ls --version
0.5.0        # make sure you have at least 0.5.0
```

## Configuration

Configure Bacon export settings with `bacon-ls` 🐽 export format and proper span support in `~/.config/bacon/prefs.toml`:

```toml
[jobs.bacon-ls]
command = [ "cargo", "clippy", "--tests", "--all-targets", "--all-features", "--message-format", "json-diagnostic-rendered-ansi" ]
analyzer = "cargo_json"
need_stdout = true

[exports.cargo-json-spans]
auto = true
exporter = "analyzer"
line_format = "{diagnostic.level}:{span.file_name}:{span.line_start}:{span.line_end}:{span.column_start}:{span.column_end}:{diagnostic.message}"
path = ".bacon-locations"
```

**NOTE: `bacon` MUST be running to generate the export locations with the `bacon-ls` job: `bacon -j bacon-ls`.**

The language server can be configured using the appropriate LSP protocol and
supports the following values:

- `locationsFile` Bacon export filename (default: `.bacon-locations`).
- `updateOnSave` Try to update diagnostics every time the file is saved (default: true).
- `updateOnSaveWaitMillis` How many milliseconds to wait before updating diagnostics after a save (default: 1000).
- `updateOnChange` Try to update diagnostics every time the file changes (default: false).

### Neovim - LazyVim

```lua
vim.g.lazyvim_rust_diagnostics = "bacon-ls"
```

### Neovim - Manual

NeoVim requires [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig/) to be configured
and [rust-analyzer](https://rust-analyzer.github.io/) diagnostics must be turned off for `bacon-ls` 🐽
to properly function.

`bacon-ls` is part of `nvim-lspconfig` from commit
[6d2ae9f](https://github.com/neovim/nvim-lspconfig/commit/6d2ae9fdc3111a6e8fd5db2467aca11737195a30)
and it can be configured like any other LSP server works best when
[vim.diagnostics.opts.update_in_insert](https://neovim.io/doc/user/diagnostic.html#vim.diagnostic.Opts)
is set to `true`.

```lua
require("lspconfig").bacon_ls.setup({
    init_options = {
        updateOnSave = true
        updateOnSaveWaitMillis = 1000
        updateOnChange = true
    }
})
```

For `rust-analyzer`, these 2 options must be turned off:

```lua
rust-analyzer.checkOnSave.enable = false
rust-analyzer.diagnostics.enable = false
```

### VSCode

The extension can be configured using the VSCode interface.

## Troubleshooting

`bacon-ls` 🐽 can produce a log file in the folder where its running by exporting the `RUST_LOG` variable in the shell:

### Neovim

```bash
❯❯❯ export RUST_LOG=debug
❯❯❯ nvim src/some-file.rs
# or
❯❯❯ RUST_LOG=debug nvim src/some-file.rs
❯❯❯ tail -F ./bacon-ls.log
```

### VSCode

Enable debug logging in the extension options.

```bash
❯❯❯ tail -F ./bacon-ls.log
```

## How does it work?

`bacon-ls` 🐽 reads the diagnostics location list generated
by [Bacon's export-locations](https://dystroy.org/bacon/config/#export-locations)
and exposes them on STDIO over the LSP protocol to be consumed
by the client diagnostics.

It requires [Bacon](https://dystroy.org/bacon/) to be running alongside
to ensure regular updates of the export locations.

The LSP client reads them as response to `textDocument/diagnostic` and `workspace/diagnostic`.

## Thanks

`bacon-ls` 🐽 has been inspired by [typos-lsp](https://github.com/tekumara/typos-lsp).
