# Contributing

go-ruby-opentype is BSD-3-Clause. The code lives at
[github.com/go-ruby-opentype/opentype](https://github.com/go-ruby-opentype/opentype).

## Ground rules

- **CGO-free.** No cgo, ever — the package must build and test on `amd64`,
  `arm64`, `riscv64`, `loong64`, `ppc64le`, `s390x`, plus `js/wasm`.
- **100 % coverage.** The CI gate enforces 100 % statement coverage, including
  error branches.
- **Backed by go-opentype.** The font parser, shaper, bidi engine and font
  registry live in [go-opentype](https://github.com/go-opentype); this repo
  is only the Ruby-idiomatic, Hash/Array-returning adapter (see [How it maps
  to go-opentype](mapping.md)). Changes to shaping/parsing/bidi behaviour
  belong upstream.

## Build & test

```sh
go build ./...
go vet ./...
go test -race -coverprofile=cover.out ./...
go tool cover -func=cover.out | tail -1   # must read 100.0%
```

## Adding a Ruby-facing method

New surface follows the existing shape:

1. Add the exported Go method to `Module`, `Font` or `Face` in `opentype.go`.
2. Return a Ruby-shaped value — a `map[string]any`, a `[]any`, or a scalar —
   never a bare Go struct.
3. `Call` and `Methods` pick it up automatically via reflection (its
   snake_case name derives from the Go method name); no dispatch table to
   update.
4. Document it in [API](api.md) and, if it changes the intended rbgo surface,
   in [Getting started in rbgo](rbgo.md).
