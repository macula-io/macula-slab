# Macula fork of `slab` 0.4.12

Vendored fork of [tokio-rs/slab](https://github.com/tokio-rs/slab) at
version `0.4.12`, with a single patch flipping the default feature set
from `["std"]` to `[]` so `no_std` consumers (via Cargo's `[patch.crates-io]`)
do not get `std` re-enabled through feature unification.

Used by [macula-kernel](https://codeberg.org/macula-internal/macula-kernel)
to satisfy `quinn-proto`'s transitive dependency on `slab` without
pulling `std` into the kernel target.

## The patch

```diff
 [features]
 std = []
-default = ["std"]
+default = []
```

Applied to both `Cargo.toml` (cargo-normalized) and `Cargo.toml.orig`
(upstream hand-written).

## Why this is needed

Upstream `slab` correctly gates `extern crate std;` and `extern crate
std as alloc;` on `feature = "std"` (and falls back to `extern crate
alloc;` when `std` is off). The crate's source compiles fine in a
`no_std` environment when `std` is off.

The problem is at the cargo-feature layer: `default = ["std"]` plus
`quinn-proto` pulling `slab` with default features unifies `std` back
on. Source-level gating is then bypassed and `extern crate std;` at
`src/lib.rs:124` plus `extern crate std as alloc;` at `src/lib.rs:126`
both emit `E0463: can't find crate for std` against the kernel target.

Flipping `default = []` makes `std` strictly opt-in.

## Upstream contribution

File an upstream PR proposing `default = []`. Upstream may resist for
backwards-compatibility reasons. If accepted, retire this fork.

## Versioning policy

Track `slab` upstream releases of 0.4.x. Re-roll by:

1. Downloading `slab-X.Y.Z.crate` tarball from crates.io.
2. Extracting into a fresh checkout.
3. Re-applying the `default = []` patch to both `Cargo.toml` and `Cargo.toml.orig`.
4. Tagging as `vX.Y.Z-macula1`.
5. Bumping the git ref in `macula-kernel`'s `Cargo.toml`.

## License

Inherited from upstream `slab` (MIT). No new code, only the Cargo.toml
feature-default flip.
