# rustCrateTemplate

A Cargo workspace template for Smiduweorc Rust projects: a library crate that
holds the program, a thin CLI crate that calls it, strict lints, tabs, and CI
on Linux, macOS and Windows. Same tasks, hooks and release flow as
[black-garden-ants][bga]. No runtime dependencies.

Not included: Deno FFI, WASM or other host glue (add it as its own crate),
and publishing (every crate starts with `publish = false`). For a JavaScript
library with a native hot path use
[nodeaddons](https://github.com/Smiduweorc/nodeaddons); for TypeScript or Go,
use black-garden-ants.

[bga]: https://github.com/Smiduweorc/black-garden-ants

## Starting from it

1. Use this template on GitHub, or
   `gh repo create <name> --template Smiduweorc/rustCrateTemplate --clone`.
2. Install [mise](https://mise.jdx.dev) and [rustup](https://rustup.rs), then
   run `just setup` (or `scripts/tasks.sh setup` before just exists). It
   installs just, lefthook, git-cliff and commitlint-rs from `mise.toml`, the
   toolchain from `rust-toolchain.toml`, the crates and the git hooks.
3. Rename `app`: the directories `crates/app` and `crates/app-cli`, then
   `rg -w app crates` for the rest (the `name` and `[[bin]]` fields, the path
   dependency, `app::` paths, and the `usage: app` and `app:` strings in
   `main.rs` and `tests/cli.rs`). Also change `CARGO_BIN_EXE_app` in
   `tests/cli.rs`, which that search misses because it is part of a longer
   name.
4. Replace `greet`. Keep the program in the library and the CLI to argument
   parsing and printing.

Run `just` to list the tasks. Tasks, hooks, commit rules and `release.sh` are
explained in [black-garden-ants][bga]; the shared files are checked byte for
byte against it by `scripts/check-shared.sh` on every push and every Monday.
Delete that script and `.github/workflows/shared.yml` if your project means to
diverge.

`unsafe_code` is forbidden in every crate, and the library denies printing to
stdout or stderr. A crate that needs `unsafe` (an FFI crate) sets its own
`[lints]` table instead of `workspace = true`.

## Known quirks

- The Rust release is pinned in two places that must agree: `channel` in
  `rust-toolchain.toml` and `rust-version` in `Cargo.toml`. Dependabot does
  not touch either; raise them together by hand.
- The edition is in both `Cargo.toml` and `rustfmt.toml`, because the
  pre-commit hook runs rustfmt on staged files without Cargo.
- rustfmt also formats the out-of-line modules a staged file declares, so the
  hook can change a file you did not stage. It stages only what was already
  staged.
- commitlint-rs publishes no binaries for its recent releases, so the first
  `just setup` builds it with cargo, which takes a while. mise does that before
  setup installs the pinned toolchain, so with `RUSTUP_AUTO_INSTALL=0` run
  `rustup toolchain install` first.
- If commitlint-rs is not on `PATH` (mise not activated in the shell that runs
  git), the commit-msg hook falls back to a regex that checks only the
  header's shape and length.
- Every cargo call passes `--locked`, so a stale `Cargo.lock` fails the task
  instead of being rewritten.
- CI does not cache `target/`. Add caching when a build is slow enough to need
  it.

## Licence

ISC, as set in `Cargo.toml`.
