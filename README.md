# Djot.lua

[![GitHub
CI](https://github.com/jgm/djot.lua/workflows/CI%20tests/badge.svg)](https://github.com/jgm/djot.lua/actions)

This repository contains a Lua parser for
[djot](https://djot.net), a light markup syntax.

Despite being written in an interpreted language, this
implementation is very fast (converting a 260K test document in
141 ms on an M1 mac using the standard `lua` interpreter). It
can produce an AST, rendered HTML, or a stream of match tokens
that identify elements by source position, which could be used
for syntax highlighting or a linting tool.

We also provide a custom pandoc writer for djot (`djot-writer.lua`),
so that documents in other formats can be converted to djot
format, and a custom pandoc reader (`djot-reader.lua`), so that
djot documents can be converted to any format pandoc supports.
To use these, just put them in your working directory and use
`pandoc -f djot-reader.lua` to convert from djot, and `pandoc -t
djot-writer.lua` to convert to djot. You'll need pandoc version
2.18 or higher, and you'll need the djot library to be installed
in your `LUA_PATH`; see [Installing](#installing), below.  If
you're using the dev version of djot or don't want to worry
about the djot library being installed, you can create
self-contained versions of the custom reader and writer
using the `amalg` tool:

    luarocks install amalg
    make djot-reader.amalg.lua
    make djot-writer.amalg.lua

These can be moved anywhere and do not require any Lua libraries
to be installed.

## Installing

To install djot using [luarocks](https://luarocks.org), just

```
luarocks install djot
```

This will install both the library and the executable `djot`.

## Using the Lua library

### Quick start

If you just want to parse some input and produce HTML:

``` lua
local djot = require("djot")
local input = "This is *djot*"
local doc = djot.parse(input)
local html = djot.render_html(doc)
```

The AST is available as a Lua table, `doc.ast`.

To render the AST:

``` lua
local rendered = djot.render_ast_pretty(doc)
```

Or as JSON:

``` lua
local rendered = djot.render_ast_json(doc)
```

To alter the AST with a filter:

``` lua
local src = "return { str = function(e) e.text = e.text:upper() end }"
local filter = djot.filter.load_filter(src)
djot.filter.apply_filter(doc, filter)
```

For a streaming parser:

``` lua
for startpos, endpos, annotation in djot.parse_events("*hello there*") do
  print(startpos, endpos, annotation)
end
```

(This will print start and end byte offsets into the input
for annotated tokens.)

## The code

The code for djot (excluding the test suite) is standard Lua,
compatible with lua 5.1--5.4 and luajit. Djot has no external
dependencies. You can run it without installing it using
`./run.sh`.

`make install` will build the rockspec and install the
library and executable using luarocks. Once installed,
the library can be used by Lua programs, and the executable can
be run using `djot`. `djot -h` will give help output.

If you can't assume that lua or luajit will be installed on
the target machine, you can use `make djot` in the `clib`
directory to create a portable binary that bakes in a lua
interpreter and the necessary scripts.

`make test` will run the tests, and `make testall` will also
run some tests of pathological cases.

## License

The code and documentation are released under the MIT license.

