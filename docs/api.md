# API

The Ruby-facing surface is three handles — `Module`, `Font`, `Face` — plus the
uniform dynamic entry point, `Call`, that an rbgo binding drives from
`method_missing`. Every method listed here is reachable through `Call` under
its snake_case name (e.g. `Font.GlyphIndex` → `glyph_index`); `Methods(recv)`
lists them at runtime.

## Module

The package-level receiver — the `Opentype` module under rbgo. Stateless,
safe for concurrent use.

| Method | Returns |
| --- | --- |
| `open_font(ttf)` | a `Font` handle, parsed from a TrueType/OpenType blob |
| `parse(ttf)` | alias for `open_font` |
| `most_legible` | the bytes of the bundled Atkinson Hyperlegible family |
| `default_font` | a `Font` handle for `most_legible` |
| `load(name)` | the bytes of a bundled family by name, or the default when empty/unknown |
| `families` | Array of Hashes `{"name", "kind", "license", "import_path"}` for all 44 families |
| `visual_order(text, base)` | `text` reordered to visual (left-to-right) order |
| `resolve_levels(text, base)` | Array of the bidi embedding level (an Int) of each rune |
| `shape(face, text, opts)` | Array of positioned glyph Hashes (see [Shape](#shape)) |

`base` is `"ltr"`, `"rtl"` or `"auto"` (also the default for an empty or
unrecognised string).

## Font

A parsed font. Immutable, safe for concurrent use.

| Method | Returns |
| --- | --- |
| `num_glyphs` | Integer glyph count |
| `glyph_index(rune)` | the glyph id (an Integer) for a code point, or `nil` |
| `axes` | Array of Hashes `{"tag", "min", "default", "max", "flags", "name_id"}` — the font's variation axes (empty for a non-variable font) |
| `named_instances` | Array of Hashes `{"subfamily_name_id", "flags", "coordinates", "post_script_name_id"}` |
| `face(px)` | a `Face` handle, the font sized at `px` pixels |

## Face

A sized font. Caches rasterised glyphs; **not** safe for concurrent use (size
one `Face` per goroutine/thread).

| Method | Returns |
| --- | --- |
| `measure(text)` | Integer advance width of `text`, in whole pixels, ignoring kerning |
| `advance(rune)` | Integer advance width of a single rune |
| `kern(prev, rune)` | Integer kerning adjustment between two adjacent runes |
| `metrics` | Hash `{"ascent", "descent", "height", "scale"}` |
| `glyph_info(rune, x, y)` | a `GlyphMask`-style Hash (see below), the glyph placed at pen `(x, y)` |
| `set_hinting(on)` | toggles grid-fitting hinting |
| `set_variation(coords)` | moves the face along its variation axes; `coords` is a Hash of axis tag → numeric user coordinate (e.g. `{"wght" => 700}`); returns the normalised coordinates as a Hash |

`glyph_info` returns:

```
{ "found"=>, "advance"=>,
  "bounds"=>{"min_x"=>, "min_y"=>, "max_x"=>, "max_y"=>, "width"=>, "height"=>},
  "origin"=>{"x"=>, "y"=>},
  "mask"=>{"width"=>, "height"=>, "stride"=>, "pix"=>} }  # present only when found
```

## Shape

`Opentype.shape(face, text, opts)` runs the HarfBuzz-lite shaper and returns
an Array of Hashes, one per glyph:

```
{ "gid"=>, "cluster"=>, "x_advance"=>, "y_advance"=>,
  "x_offset"=>, "y_offset"=>, "scale"=> }
```

`opts` is a Hash (or `nil`) carrying:

| Key | Meaning |
| --- | --- |
| `"direction"` | `"ltr"` / `"rtl"` / `"auto"` |
| `"script"` | an ISO 15924 tag, e.g. `"arab"`, `"deva"`, `"thai"`, `"khmr"`, `"egyp"`, `"hang"` |
| `"features"` | an Array of OpenType feature tags |
| `"vertical"` | a Bool — shape for vertical text flow |

A `nil` face yields an empty Array.

## Bidi

`visual_order` and `resolve_levels` on `Module` implement the Unicode
Bidirectional Algorithm (UAX #9) directly — mixed left-to-right/right-to-left
text (e.g. Latin mixed with Arabic or Hebrew) resolves without any
`golang.org/x/text` dependency.

## Fonts

`families`, `load`, `most_legible` and `default_font` on `Module` expose the
[go-opentype/fonts](https://github.com/go-opentype/fonts) registry: 44
bundled, legible, permissively-licensed families, with Atkinson Hyperlegible
embedded directly as the default.

## Dynamic dispatch

```
Call(recv, method, *args) -> result
Methods(recv)             -> Array<String>
```

`Call` reflects `method` (snake_case) to the matching exported Go method on
`recv` (a `Module`, `Font` or `Face`), coercing each argument: Ruby
Integers/Floats to Go ints or runes, Strings to Go strings or `[]byte`,
Hashes to `map[string]any`, `nil` to the zero value of pointer/slice/map
parameters. A trailing Go `error` return is unwrapped into `Call`'s own
error. This is the single entry point an rbgo `method_missing` binding
drives — see [Getting started in rbgo](rbgo.md).
