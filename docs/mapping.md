# How it maps to go-opentype

`go-ruby-opentype/opentype` does not reimplement any text-stack logic. It is
a thin adapter: every handle wraps a value from a
[go-opentype](https://github.com/go-opentype) package and every method is a
one- or two-line call into it, reshaping the result into a Ruby-shaped value.

| go-ruby-opentype | go-opentype | What changes at the boundary |
| --- | --- | --- |
| `Module.OpenFont` / `.Parse` | [`opentype.Parse`](https://github.com/go-opentype/opentype) | wraps the returned `*opentype.Font` in a `Font` handle |
| `Module.MostLegible` / `.Load` / `.Families` | [`fonts.MostLegible`, `fonts.All`](https://github.com/go-opentype/fonts) | family metadata (`Kind`, an enum) becomes a Hash with a stringified `"kind"` |
| `Module.VisualOrder` / `.ResolveLevels` | [`bidi.VisualOrder`, `bidi.ResolveLevels`](https://github.com/go-opentype/bidi) | a `base` string (`"ltr"`/`"rtl"`/`"auto"`) maps to `bidi.Direction`; levels (`[]bidi.Level`) become an `Array` of Ints |
| `Module.Shape` | [`shape.Shape`](https://github.com/go-opentype/shape) | an `opts` Hash builds a `shape.Options`; each `shape.Glyph` becomes a 7-key Hash |
| `Font.NumGlyphs` / `.GlyphIndex` | `(*opentype.Font).NumGlyphs`, `.GlyphIndex` | `GlyphIndex`'s `(gid, ok)` pair becomes `gid` or `nil` |
| `Font.Face` | `(*opentype.Font).NewFace` | wraps the returned `*opentype.Face` in a `Face` handle |
| `Font.Axes` / `.NamedInstances` | `(*opentype.Font).Axes`, `.NamedInstances` | each struct becomes a Hash; a `Coordinates map[string]float64` stays a Hash |
| `Face.Measure` / `.Advance` / `.Kern` | `(*opentype.Face).Measure`, `.Advance`, `.Kern` | direct pass-through, no reshaping |
| `Face.Metrics` | `(*opentype.Face).Metrics` + `.Scale` | struct fields plus a separately-fetched `Scale` become one Hash |
| `Face.GlyphInfo` | `(*opentype.Face).GlyphMask` | the `(bounds, mask, origin, advance, ok)` tuple becomes a nested Hash; `mask` (an `*image.Alpha`) is present only when found, as `{width, height, stride, pix}` |
| `Face.SetHinting` / `.SetVariation` | `(*opentype.Face).SetHinting`, `.SetVariation` | `SetVariation`'s coordinate Hash is filtered to numeric values and echoed back normalised |

## What stays entirely upstream

- **Parsing** (`cmap`, `glyf` TrueType outline decoding) and **rasterisation**
  (4×4 supersampled anti-aliasing) — [`go-opentype/opentype`](https://github.com/go-opentype/opentype).
- **Shaping** — cursive joining, ligatures, mark attachment, kerning, script
  rules for Arabic, Indic/USE, Thai, Khmer, Egyptian hieroglyphs, Hangul and
  vertical text — [`go-opentype/shape`](https://github.com/go-opentype/shape).
- **The Bidirectional Algorithm itself** (UAX #9 rule resolution) —
  [`go-opentype/bidi`](https://github.com/go-opentype/bidi).
- **The bundled font bytes and licenses** —
  [`go-opentype/fonts`](https://github.com/go-opentype/fonts).

Nothing in `go-ruby-opentype/opentype` parses a font file, walks a `glyf`
table, runs a shaping rule, or resolves a bidi level on its own; it only
translates typed Go values at the boundary. Bug reports about *shaping
correctness*, *rasterisation quality* or *bidi conformance* belong upstream in
go-opentype; reports about *argument coercion*, *the Hash/Array shape of a
result*, or *`Call`/`Methods` dispatch* belong here.

## Why the split

This mirrors every other `go-ruby-*` adapter over a typed Go library — see
[`go-ruby-dimail/dimail`](https://github.com/go-ruby-dimail/dimail) over
[`go-dimail`](https://github.com/go-dimail), or
[`go-ruby-regexp/regexp`](https://github.com/go-ruby-regexp/regexp) over the
pure-Go Onigmo engine: the typed library stays reusable from plain Go and
from any other language binding, while this repo carries only the
Ruby-idiomatic reshaping that [go-embedded-ruby](https://github.com/go-embedded-ruby)
(rbgo) needs.
