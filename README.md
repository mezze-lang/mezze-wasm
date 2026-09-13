# mezze-wasm

WebAssembly interop for [Mezze](https://mezze-lang.org) — packaged as
an external `mez` dependency (polyglot-as-a-package Phase 6, mirroring
[mezze-python](https://github.com/mezze-lang/mezze-python) and
[mezze-js](https://github.com/mezze-lang/mezze-js)).

## Install

```toml
# your Mezze.toml

[dependencies]
mezze_wasm = "github:mezze-lang/mezze-wasm@0.1.0"

[polyglot.wasm]
# Optional: URLs of `.wasm` modules to fetch. Each entry lands under
# `.deps/wasm/<basename>.wasm`. Leave empty to vendor `.wasm` files
# in the repo and load them by path directly.
packages = []
```

Then `mez sync` and you can:

```mezze
use mezze_wasm::wasm::{ Wasm, GraalWasm, WasmForeign, from_int }

pub let main = {} -> perform GraalWasm in do
  let mod_r = Wasm.load_file { path = "demo.wasm", name = "demo" }
  match mod_r is
    Ok { val = module } -> match module.call_export { name = "add", args = [from_int { val = 2 }, from_int { val = 3 }] } is
      Ok { val = f } -> match f.as_int{} is
        Ok { val = n } -> ...
        Err { error = _ } -> ...
      Err { error = _ } -> ...
    Err { error = _ } -> ...
```

## What ships

- `src/wasm.mz` — `effect Wasm`, `impl Wasm for GraalWasm`, inherent
  methods on `WasmForeign` (wasm-idiomatic + universal InteropLibrary
  vocabulary), language-tagged numeric constructors.
- `src/codec.mz` — `WasmC` codec + tier-4 `ToWasm` / `FromWasm`
  sugar. Scalar-only: strings/records/lists/etc. don't cross the wasm
  value boundary — the caller marshals them through linear memory.
- `[polyglot.provider]` — a `curl` URL-fetch install template used by
  `mez sync` when a user project declares `[polyglot.wasm] packages`.
  Fetched `.wasm` files land under `<dest>/`; the caller loads them
  with `Wasm.load_file { path = ... }`. Users who vendor `.wasm`
  binaries locally skip the packages list entirely — nothing runs.
- `[maven]` — resolves the GraalWasm runtime jar
  (`org.graalvm.wasm:wasm-language`) from Maven Central plus the
  native-handler jar (`dev.mezze:polyglot-wasm-natives`) from
  [mezze-lang/mezze-maven](https://github.com/mezze-lang/mezze-maven).
  Aether pulls the full transitive graph — no manual sha
  bookkeeping.

## Requirements

- `mez` (Mezze CLI) 0.1 or later.
- GraalVM 25.0.3.
- `curl` on PATH iff the consuming project declares
  `[polyglot.wasm].packages` — `mez sync` runs the provider's install
  template into the local `.deps/wasm/` directory. No packages, no
  curl needed (vendored `.wasm` files work out of the box).

## License

MIT — see `LICENSE`.
