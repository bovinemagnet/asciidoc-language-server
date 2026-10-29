# AsciiDoc Language Server for Zed

A Zed extension that runs [`adoc-ls`](https://github.com/bovinemagnet/AdocFoundry/tree/main/crates/adoc-ls), a language server for AsciiDoc with Antora support, on AsciiDoc files.

It provides only the language server. Syntax highlighting, the outline and snippets come from the [AsciiDoc](https://github.com/dunyakirkali/zed-asciidoc) extension, which must be installed too: `adoc-ls` attaches to the `AsciiDoc` language it registers.

## Features

- Diagnostics for unresolved xrefs, anchors, includes and images, duplicate anchors, and Antora problems such as unknown modules, malformed resource ids and unresolved navigation entries.
- Completion of xref, include and image targets, anchors and attribute names.
- Hover, Go to Definition, Find References and renaming, including rewriting the xrefs and nav entries that name a renamed page.
- Document and workspace symbols.
- Antora resolution across components, versions and modules.
- HTML preview through code actions.

The [`adoc-ls` README](https://github.com/bovinemagnet/AdocFoundry/blob/main/crates/adoc-ls/README.md) describes these, its settings and its diagnostic codes in full.

## The language server

The extension uses an `adoc-ls` on `PATH` when there is one. Otherwise it downloads `adoc-ls` from the latest [AdocFoundry release](https://github.com/bovinemagnet/AdocFoundry/releases) for Linux, macOS and Windows on x86-64 and AArch64, and keeps it up to date. If the release cannot be fetched, it uses its most recent earlier download.

To use another binary, set it in Zed's settings:

```json
{
  "lsp": {
    "adoc-ls": {
      "binary": {
        "path": "/path/to/adoc-ls",
        "arguments": ["--stdio"]
      }
    }
  }
}
```

To build `adoc-ls` yourself:

```sh
cargo install --git https://github.com/bovinemagnet/AdocFoundry adoc-ls
```

## Development

Load this repository as a dev extension in Zed (Extensions › Install Dev Extension). Building it needs rustup with the `wasm32-wasip2` target.

```sh
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test
cargo check --target wasm32-wasip2
```

## Licence

Apache-2.0.
