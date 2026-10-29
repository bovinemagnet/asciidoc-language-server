# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Zed editor extension, written in Rust and compiled to WebAssembly, whose only job is to launch the [`adoc-ls`](https://github.com/bovinemagnet/AdocFoundry/tree/main/crates/adoc-ls) language server for AsciiDoc files. The language server itself lives in the separate AdocFoundry repository; this repository contains no LSP logic, grammars, highlighting or snippets. Those come from the separate `zed-asciidoc` extension, which registers the `AsciiDoc` language that `extension.toml` attaches `adoc-ls` to.

## Commands

Requires rustup with the `wasm32-wasip2` target (`rustup target add wasm32-wasip2`). These are the same checks CI runs (`.github/workflows/ci.yml`):

```sh
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test
cargo check --target wasm32-wasip2
```

Run a single test with `cargo test <test_name>`, e.g. `cargo test newest_version_dir_compares_versions_numerically`.

To try the extension, load the repository in Zed via Extensions › Install Dev Extension. Zed builds `extension.wasm` (gitignored) itself.

## Architecture

Everything is in `src/lib.rs`. The crate is built as both `cdylib` (the Wasm extension Zed loads) and `rlib` (so `cargo test` can run the unit tests natively on the host).

`language_server_command` resolves the `adoc-ls` binary in this order:

1. `lsp.adoc-ls.binary.path` from the user's Zed settings.
2. `adoc-ls` on the worktree's `PATH` (`worktree.which`).
3. A download from the latest non-prerelease GitHub release of `bovinemagnet/AdocFoundry`, unpacked into `adoc-ls-<version>/` in the extension's working directory. After a successful new download, other `adoc-ls-*` directories are deleted; a failed download removes its partial directory.
4. If the release cannot be fetched, the newest `adoc-ls-<version>/` already on disk (versions compared numerically).

The resolved path is cached on the extension struct for the session.

Command construction (`server_command`): arguments from `lsp.adoc-ls.binary.arguments` *replace* the default `--stdio` rather than being appended; environment from `lsp.adoc-ls.binary.env` is layered over the worktree's shell environment, overriding duplicate keys.

### Release asset contract

`release_asset` must match the asset names published by AdocFoundry's release workflow: `adoc-ls-<version>-<target>.<ext>`, where the tag's leading `v` is stripped and `<target>` is one of `linux-x86_64`, `linux-aarch64`, `macos-universal` (used for both Mac architectures), `windows-x64` or `windows-arm64`. Linux and macOS assets are `.tar.gz`; Windows assets are `.zip`. Unsupported platforms return an error pointing users at `cargo install --git https://github.com/bovinemagnet/AdocFoundry adoc-ls`. If AdocFoundry changes its asset naming, this function and its tests must change with it.

### Testing constraints

Zed host functions (`latest_github_release`, `download_file`, `current_platform`, installation status, etc.) are only available inside Zed's Wasm runtime, so they cannot be exercised by `cargo test`. Keep decision logic in pure helper functions (as `release_asset`, `newest_version_dir`, `server_command` and friends are) and test those; the existing tests live in the `tests` module at the bottom of `src/lib.rs`.

## Conventions

- `zed_extension_api` is pinned to an exact version (`=0.7.0`); bumping it can change the host API and the required Zed version.
- `unsafe_code` is forbidden via `[lints.rust]`.
- The version is declared in both `Cargo.toml` and `extension.toml`; keep them in sync.
- `.editorconfig`: 4-space indent for Rust, 2-space for Markdown, TOML, YAML and JSON.
