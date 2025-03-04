# fzf.kak

[![GitHub release][1]][2] [![GitHub Release Date][3]][4]
![Github commits (since latest release)][5] ![license][6]

**fzf.kak** is a plugin for [Kakoune][7] editor, that provides integration with the [fzf][8] tool.
There's also a limited support for [skim][9], which can be enabled by changing `fzf_implementation` option.

![showcase][10]


## Installation


### With [plug.kak][11]

The recommended way to install **fzf.kak** is to use plug.kak plugin manager.
To install **fzf.kak** add this to the `kakrc`:

``` kak
plug "andreyorst/fzf.kak"
```

Then reload the configuration file or restart Kakoune and run `:plug-install`.


### Without plugin manager

This plugin consists of several parts which are referred here as "modules".
So, for plugin to work the base module must be loaded:

``` sh
source "/path/to/fzf.kak/rc/fzf.kak" # loading base fzf module
```

This module doesn't do anything on it's own.
Each module in the `modules` directory provides features that extend base `fzf` module with new commands and mappings.
Those can be loaded manually one by one the same way as the base module, or with the use of the `find` command:

``` sh
evaluate-commands %sh{
    find -L "path/to/fzf.kak/modules/" -type f -name '*.kak' -exec printf 'source "%s"\n' {} \;
}
```


## Usage

**fzf.kak** provides a `fzf-mode` command that can be mapped to preferred key:

```kak
map global normal <c-p> ': fzf-mode<ret>'
```

This will invoke the user mode, which contains mnemonic keybindings for each sub-module.
If all modules were loaded, the following mappings are available:

- <kbd>b</kbd> - Select buffer.
- <kbd>c</kbd> - Switch server's working directory.
- <kbd>f</kbd> - Search for file and open it.
- <kbd>v</kbd> - Edit file in version control system tree.
- <kbd>Alt</kbd>+<kbd>v</kbd> - Explicitly select which VCS command to run.
- <kbd>s</kbd> - Search over buffer contents and jump to result line.
- <kbd>t</kbd> - Browse ctags tags.
- <kbd>Alt</kbd>+<kbd>t</kbd> - Select tag kind filter on per language basis.
- <kbd>g</kbd> - Interactive grep.
- <kbd>p</kbd> - Project selector.
- <kbd>Alt</kbd>+<kbd>p</kbd> - Project related commands.

When Kakoune runs inside Tmux, **fzf.kak** will use bottom split to display `fzf` window.
Additional keybindings are also made available to open file in vertical or horizontal split.
Otherwise, the `terminal` command is being used to create new windows.


## Configuration

**fzf.kak** features a lot of settings via options that can be altered to change how **fzf.kak** behaves.
Each `.kak` file provides a Kakoune module, so any settings which alter variable values should be wrapped in `ModuleLoaded` hook.
See [plug.kak manual section for `defer`](https://github.com/andreyorst/plug.kak/tree/master#deferring-plugin-configuration) that explains how to do this when using plug.kak.


### Default query

| module |
|--------|
| `fzf`  |

By default main selection is used as the default query for `fzf`, but only if the selection more than 1 character long.
This can be disabled by setting `fzf_use_main_selection` to `false`.


### Windowing

| module |
|--------|
| `fzf`  |

When using Tmux fzf.kak automatically creates all needed Tmux splits and panes for you.
In other cases **fzf.kak** uses `fzf_terminal_command` option to call windowing command to create new windows.
By default it is set to use `terminal` alias: `terminal kak -c %val{session} -e "%arg{@}"`, but some terminals can provide other aliases or commands, like `terminal-tab` in Kitty.


### Mappings

| module |
|--------|
| `fzf`  |

Keys that are used in the `fzf` window can be configured with these options:

- `fzf_window_map` - mapping to perform an action in new window,
- `fzf_vertical_map` - mapping to perform an action in new vertical split (Tmux),
- `fzf_horizontal_map` - mapping to perform an action in new horizontal split.

These options should be set to work with fzf `--expect` parameter, so check out fzf documentation on this.


### File command

| module     |
|------------|
| `fzf-file` |

A command that is used to search for files and it's arguments can be configured by changing the value of `fzf_file_command` variable, that is available in the `fzf-file` module.

Supported tools are [GNU Find][12], [The Silver Searcher][13], [ripgrep][14], [fd][15].
Default set of arguments is provided for each of these searchers, only the name of the tool can be assigned to the `fzf_file_command` variable:

```kak
set-option global fzf_file_command 'rg' # 'ag', 'fd', or 'find'
```

Default arguments can be changing by setting the complete command to execute:

```kak
set-option global fzf_file_command "find . \( -path '*/.svn*' -o -path '*/.git*' \) -prune -o -type f -print"
```


### Preview

| module     |
|------------|
| `fzf-file` |

**fzf.kak** tries to automatically detect where to show preview window, depending on aspect ratio of the new terminal window.
By default if the doubled height is bigger than the width, preview occupies upper 60% of space.
If height is smaller than the width, preview is shown at the right side.
These amounts can be configured with `fzf_preview_height` and `fzf_preview_width` options.

When using **fzf.kak** inside `tmux`, bottom pane is used for all `fzf` commands, and preview window is displayed on the right side.
When preview is turned on, height of `tmux` split is increased to provide more space.
Split height can be configured with the `fzf_preview_tmux_height` variable.

Amount of lines in preview window can be changed with `fzf_preview_lines` option.

Preview feature can be disabled entirely by setting `fzf_preview` option to `false`.


#### Highlighting preview window

| module     |
|------------|
| `fzf-file` |

Contents of the file displayed within preview window can be syntax highlighted.
This can be enabled by specifying highlighter to use with the `fzf_highlight_command` option.
These highlighters are are supported out of the box:

* [Bat][16]
* [Coderay][17]
* [Highlight][18]
* [Rouge][19]

Although other tools are not supported by the script, then should work fine as long as they work with `fzf`.


### VCS

| module    |
|-----------|
| `fzf-vcs` |

This script supports these version control systems: Git, Subversion, GNU Bazaar, and Mercurial.
By default <kbd>v</kbd> mapping from `fzf` mode will detect your version control system  automatically.
To explicitly use some particular VCS command, the <kbd>Alt</kbd>+<kbd>v</kbd> mapping can be used, which includes all supported VCS shortcuts.

To set parameters to VCS command used to provide project files the following options can be used:

* `fzf_git_command`
* `fzf_svn_command`
* `fzf_bzr_command`
* `fzf_hg_command`

Other VCS are not supported officially.
Feature requests and merge requests are welcome.


### Tmux

| module |
|--------|
| `fzf`  |

When running inside Tmux, `fzf` will use bottom split.
Height of this split can be changed with the `fzf_tmux_height` option.
`fzf_tmux_height_file_preview` option is used to control height of the split when file-preview is turned on.


### Projects

| module        |
|---------------|
| `fzf-project` |

**fzf.kak** has basic project manipulation capabilities.

To store projects a hidden file is created in `%val{config}` and called `.fzf-projects`.
The location of this file and its name can be changed by modifying the `fzf_project_file` option.
By default project paths which start from home directory will use `~` instead of read path.
To change this, set `fzf_project_use_tilda` option to `false`.


## `fzf` command

`fzf` command can be used from prompt mode and for [scripting][20].
The following arguments are supported:

- `-kak-cmd`: A Kakoune command that is applied to `fzf` resulting value, e.g. `edit -existing`, `change-directory`, e.t.c.
- `-multiple-cmd`: A Kakoune command that is applied when multiple items selected to every item but the first one.
- `-items-cmd`: A command that is used as a pipe to provide list of values to `fzf`.
  For example, if we want to pass list of all files recursively in current directory, we would use `-items-cmd %{find .}` which will be piped to `fzf` tool.
- `-fzf-impl`: Override `fzf` implementation variable.
  Can be used if command needs to provide a different arguments to `fzf`.
  See [sk-grep.kak][21] as example.
- `-fzf-args`: Additional flags for `fzf` program.
- `-preview-cmd`: A preview command.
  Can be used to override default preview handling.
- `-preview`: If specified, command will ask for preview.
- `-filter`: A pipe which will be applied to result provided by `fzf`.
  For example, if we are returning such line `3 hello, world!` from `fzf`, and we are interested only in the first field which is `3`, we can use `-filter %{cut -f 1}`.
  Basically everything what `fzf` returns is piped to this filter command.
  See [fzf-search.kak][22] as example.
- `-post-action`: Extra commands that are preformed after `-kak-cmd` command.


## Contributing

Please do.
If you want to contribute to **fzf.kak** by adding a module, you can submit one by providing a pull request, or just open a feature request and we'll see what can be done.

The basic idea behind the module structure can be described as:

1. Provide a user module;
2. Define a command that calls `fzf` function with appropriate arguments;
3. Create a mapping in a `ModuleLoaded` hook, that requires new module, and calls the command.

See how existing modules are implemented to understand the idea of how modules are constructed.


### External modules

Support for [yank-ring.kak][25] was externalized to separate plugin [fzf-yank-ring.kak][24]

[1]: https://img.shields.io/github/release/andreyorst/fzf.kak.svg
[2]: https://github.com/andreyorst/fzf.kak/releases
[3]: https://img.shields.io/github/release-date/andreyorst/fzf.kak.svg
[4]: https://github.com/andreyorst/fzf.kak/releases
[5]: https://img.shields.io/github/commits-since/andreyorst/fzf.kak/latest.svg
[6]: https://img.shields.io/github/license/andreyorst/fzf.kak.svg
[7]: https://github.com/mawww/kakoune
[8]: https://github.com/junegunn/fzf
[9]: https://github.com/lotabout/skim
[10]: https://user-images.githubusercontent.com/19470159/46813471-6ee76800-cd7f-11e8-89aa-123b3a5f9f1b.gif
[11]: https://github.com/andreyorst/plug.kak
[12]: https://www.gnu.org/software/findutils/
[13]: https://github.com/ggreer/the_silver_searcher
[14]: https://github.com/BurntSushi/ripgrep
[15]: https://github.com/sharkdp/fd
[16]: https://github.com/sharkdp/bat
[17]: https://github.com/rubychan/coderay
[18]: https://gitlab.com/saalen/highlight
[19]: https://github.com/jneen/rouge
[20]: #writing-a-module
[21]: rc/modules/sk-grep.kak
[22]: rc/modules/fzf-search.kak
[23]: #configuration
[24]: https://gitlab.com/losnappas/fzf-yank-ring.kak
[25]: https://github.com/alexherbo2/yank-ring.kak

<!--  LocalWords:  Github Kakoune fzf kak config VCS ctags Tmux fd sk
      LocalWords:  ripgrep readme Coderay rc
 -->
