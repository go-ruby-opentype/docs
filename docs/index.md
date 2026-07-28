# go-ruby-opentype

The pure-Go, Ruby-runtime-independent core of the Ruby **`opentype`** gem — a
complete text stack (font parsing, sized faces, complex-script shaping, the
Unicode Bidirectional Algorithm and a registry of legible fonts), shaped so
that [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) (rbgo) can
bind it as `require "opentype"`.

It is a thin adapter over the typed libraries of the
[go-opentype](https://github.com/go-opentype) stack:

| Library | Role |
| --- | --- |
| [`go-opentype/opentype`](https://github.com/go-opentype/opentype) | TrueType/OpenType parser + anti-aliased rasteriser |
| [`go-opentype/shape`](https://github.com/go-opentype/shape) | HarfBuzz-lite complex-text shaper (Arabic, Indic, USE, Thai, Khmer, Egyptian, Hangul, vertical text, …) |
| [`go-opentype/bidi`](https://github.com/go-opentype/bidi) | The Unicode Bidirectional Algorithm (UAX #9) |
| [`go-opentype/fonts`](https://github.com/go-opentype/fonts) | A registry of 44 legible, permissively-licensed families |

It exposes them through three Ruby-facing handles — `Module`, `Font`, `Face`
— whose methods return **Ruby-shaped values**: a **Hash** (`map[string]any`),
an **Array** (`[]any`) or a scalar. A single dynamic entry point, `Call`,
dispatches a Ruby-style snake_case method name to the matching handle method
and coerces the arguments, which is exactly what an rbgo binding drives from
`method_missing`. Nothing here depends on the Ruby runtime, so it is equally
usable as a standalone Go library — a sibling of `go-ruby-regexp/regexp`,
`go-ruby-erb/erb` and `go-ruby-dimail/dimail`.

## At a glance

- **CGO-free** — builds and tests identically on `amd64`, `arm64`, `riscv64`,
  `loong64`, `ppc64le`, `s390x`, plus `js/wasm`.
- **100 % test coverage**, race-clean, enforced in CI.
- **No `HarfBuzz`, no `FreeType`, no `x/image`, no `x/text`** — the entire
  stack, from sfnt parsing to complex-script shaping, is stdlib-only Go.
- Backed by [go-opentype](https://github.com/go-opentype) (the typed engine,
  shaper, bidi implementation and font registry).

## Install

```sh
go get github.com/go-ruby-opentype/opentype
```

See [API](api.md) for the full Ruby-facing surface, [Getting started in
rbgo](rbgo.md) for the `require "opentype"` workflow, and [How it maps to
go-opentype](mapping.md) for how each handle relates to the underlying
libraries.
