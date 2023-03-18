snippets plugin for lite-xl


### Installation

Copy the files into the editor's `plugins` directory.

* `snippets.lua`: the base plugin which includes features such as snippet
	expansion, tabbing through tabstops, etc.
* `lsp_snippets.lua`: requires the base plugin; adds support for lsp/vscode style snippet.
* `json.lua`: [rxi's json libray](https://github.com/rxi/json.lua), required to
	load LSP snippets from json files. `lsp_snippets` will attempt to load it from
	lint+ or the lsp plugin if they're in the plugin path, so it is only needed
	if neither can be found.


### Examples

Adding an LSP style snippet with a `fori` trigger for lua files:

```lua
snippets.add {
	trigger  = 'fori',
	files    = '%.lua$',
	info     = 'ipairs',             -- optional, used by the autocomple menu
	desc     = 'numerical for loop', -- optional, used by the autocomple menu
	format   = 'lsp',                -- 'lsp' must be lowercase
	template = [[
for ${1:i}, ${2:v} in ipairs($3) do
	$0
end
]]
}
```

Adding LSP snippets from json files:

```lua
local lsp_snippets = require 'plugins.lsp_snippets'
lsp_snippets.add_paths {
	'snippets',                   -- relative paths are prefixed with the userdir
	'/path/to/snippets/folder',
	'/specific/snippet/file.json'
}
```

Executing a snippet in the current doc at each cursor:

```lua
snippets.execute {
	format   = 'lsp',
	template = [[
local function $1($2)
	$0
end
]]
}
```


### Simple usage

#### Commands

`snippets:next`: sets selections for the next tabstop. (wraps around to 1)

`snippets:previous`: sets selections for the previous tabstop. (wraps around to the last tabstop)

`snippets:select-current`: selects the values of the current tabstop.

`snippets:exit`: sets selections for either the end tabstop (e.g `$0`) or after the snippet.

`snippets:next-or-exit`: if the current tabstop is the last one, exits; otherwise, next.

By default, `snippets:next-or-exit` is bound to `tab`, and `snippets:previous`
to `shift+tab`.

#### API

`snippets.add(snippet)` -> `(id, ac | nil) | nil`
* `snippet`: the snippet to add. Schema:
	* `template`: the snippet template (e.g `'$1 some text ${2:and more text}'`)
	* `format`: the format of the template (e.g `lsp`).
	  If unspecified, the template is directly inserted as plain text.
	* `nodes`: it is also possible to directly use nodes, which are the internal
	  representation of a snippet. If the given snippet contains neither a template
	  nor nodes, this function is a no op and returns `nil`.
	* `defaults`, `transforms`, `choices`, `matches`: (optional) See [docs.md](docs.md)
	* `trigger`: if the autocomplete plugin is enabled, the given snippet will
	  be added as a completion item with this trigger.
	* `files`: when the autocompletion should be enabled. e.g `'%.lua$'` for lua files.
	* `info`: (optional) the name on the right of the trigger in the autocompletion menu.
	* `desc`: (optional) the description that shows up in the autocompletion menu;
	  defaults to the template, if any.

* `id`: this function returns an id which can be reused to execute this exact snippet.
	If the snippet is parsed from a template, it will be parsed only once if executed
	with this id.
* `ac`: if the snippet was added as an autocompletion item, this function also returns
	said item.

`lsp_snippets.add_path(paths)`
* `paths`: a single path or an array of paths.
	* if a path is a relative path, then it is prefixed with the userdir, e.g
	`snippets` -> `~/.config/lite-xl/snippets` if the userdir is `~/.config`.
	* if it is a file, then it is added only if it has a valid file name in the
		form of `languagename.json` (case is ignored).
	* if it is a folder, then:
		* all files with a valid name are added;
		* subfolders with a language name have all their json files added,
			regardless of their name (e.g `python/main.json`). This is not recursive,
			e.g `python/python/main.json` will not work.

`snippets.execute(snippet, doc, partial)` -> `true | nil`
* `snippet`: the snippet to expand; it will be inserted at each cursor in `doc`.
	This is either an id returned from `snippets.add` or a snippet as would be
	given to `snippets.add`.
* `doc`: the doc in which to expand the snippet; if nil, the current doc is used.
* `partial`: if truthy, remove the 'partial symbol', e.g the current selection or
	the trigger if expanded from an autocompletion.

* this function returns `true` if it successfully completed; `nil` otherwise.


### Advanced

See [docs.md](docs.md)
