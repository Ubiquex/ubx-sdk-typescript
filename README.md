# @ubx/sdk

The describe-only TypeScript runtime for [Ubiquex](https://github.com/ubiquex/ubiquex) SDK programs.

A program built on `@ubx/sdk` never computes, never reaches a provider,
and never touches a ledger -- it describes a desired end-state (an
`ubx:intent/v1` document) and stops. `resource()` returns a `Computed<T>`
reference, never a real value, so a resource's not-yet-known attribute
can still be wired into a sibling resource's config at describe time.

This package is the runtime shared by every `ubx sdk gen --lang ts`
generated bindings package (e.g. `ubx-sdk-aws`, `ubx-sdk-google`,
`ubx-sdk-azure`, `ubx-sdk-kubernetes` -- one combined repo per provider,
UBI-138) -- a program built against it imports `stack`/`resource`/
`intent`/`secret` and calls them against the binding's own generated
`ResourceBinding`/`Config` types.

`evaluator/guards.ts` is the eager-nondeterminism guard set the
hermetic evaluator installs before running a program (`Date.now`,
`Math.random`, network/filesystem access, ...) -- not something a
program author imports directly.

## Two real roles, one source (UBI-139)

This repo is the canonical source for both:

- The real, published JSR package (`deno add jsr:@ubx/sdk`, or
  `jsr:@ubx/sdk` as an import specifier) -- an independent convenience
  for a program author's own editor/IDE type-checking, never required
  for evaluation to work.
- The runtime `ubx`'s own hermetic Deno-based evaluator actually
  executes against -- `github.com/ubiquex/ubiquex` mounts this repo as
  a git submodule at `sdk/ts/`, and `embed.go`'s own `go:embed`
  directive compiles `evaluator/guards.ts` and `runtime/src/index.ts`
  directly into the `ubx` binary itself (`tseval` extracts both to a
  temp dir and generates a `deno.json` import map at evaluation time --
  the real `jsr:@ubx/sdk` package is never fetched or consulted; `ubx`'s
  own hermetic evaluator runs under Deno's `--no-remote`, which
  structurally refuses to resolve any remote specifier, jsr: included).

Moving or renaming `evaluator/guards.ts` or `runtime/src/index.ts`
within this repo requires a matching update to `ubiquex`'s own
`sdk/ts/embed.go`, or the whole `ubx` binary fails to build.

See [docs.ubiquex.io](https://docs.ubiquex.io) for the full SDK guide.

## License

Apache-2.0
