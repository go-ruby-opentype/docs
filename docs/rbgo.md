# Getting started in rbgo

!!! note "Status"
    The `require "opentype"` binding lives in
    [go-embedded-ruby/ruby](https://github.com/go-embedded-ruby/ruby) (rbgo)
    and is **pending** in that repo. This page documents the intended
    surface — everything below is exactly what `go-ruby-opentype/opentype`
    already provides today via [`Call`](api.md#dynamic-dispatch); only the
    thin `method_missing` shim inside rbgo is outstanding.

## Why a separate adapter repo

`go-ruby-opentype/opentype` has **no import of rbgo or the Ruby runtime**. It
is a standalone Go library whose handles (`Module`, `Font`, `Face`) already
speak in Ruby-shaped values — Hashes, Arrays, scalars — and whose `Call`
function is the exact uniform surface a `method_missing` shim needs. rbgo's
job, once wired, is only to:

1. register an `Opentype` Ruby module backed by `opentype.NewModule()`;
2. route `Opentype.foo(*args)` to `opentype.Call(mod, "foo", args...)`;
3. wrap the `Font`/`Face` handles returned by `open_font`/`face` as Ruby
   objects whose own `method_missing` also routes through `Call`.

## Intended usage

```ruby
require "opentype"

# Module-level calls resolve through Opentype.
font = Opentype.open_font(Opentype.most_legible)
puts font.num_glyphs                       # => Integer

face = font.face(24)                       # Font#face -> a Face object
puts face.measure("Hello")                 # => Integer

# Complex-script shaping: Arabic cursive joining + bidi, in one call.
Opentype.shape(face, "بيت").each do |g|    # => Array<Hash>
  # draw glyph g["gid"] at the pen position implied by g["x_advance"], etc.
end

# Mixed-direction text reordered for left-to-right display.
puts Opentype.visual_order("aب1", "auto")

# Every bundled family, by name.
Opentype.families.each { |f| puts f["name"] }
```

## Argument and result shapes

| Ruby side | Go side |
| --- | --- |
| Integer / Float | `int`, `int64`, or a `rune` (single-character coercion) |
| String | `string` or `[]byte` (font blobs) |
| Hash | `map[string]any` (e.g. `shape` opts, `set_variation` coords) |
| `nil` | the zero value — an omitted trailing argument, an unset pointer/slice/map |
| result: single record | a Ruby Hash |
| result: a list | a Ruby Array of Hashes |
| result: a count/flag/string | a plain scalar |

Any error from the underlying Go call raises inside `Call` and should surface
as a Ruby exception once wired into rbgo's error-mapping convention.

## Standalone Go usage today

Everything above is already exercised, without Ruby, in the
[README](https://github.com/go-ruby-opentype/opentype#usage-from-go) and in
[`example_test.go`](https://github.com/go-ruby-opentype/opentype/blob/main/example_test.go):

```go
m := opentype.NewModule()
font, _ := m.OpenFont(m.MostLegible())
face := font.Face(24)
run := m.Shape(face, "AV", nil) // an Array of Hashes
```

Follow [go-embedded-ruby/ruby](https://github.com/go-embedded-ruby/ruby) for
the binding's landing; once it ships, `require "opentype"` will need no code
change here.
