# Google's Rust Crate Audits

Google uses cargo-vet to ensure third-party Rust dependencies have been audited
by Googlers or other trusted entities.

This repository automatically aggregates Google's audits from various
repositories to make them easily reusable by others.

To import Google's audits into another cargo-vet instance, add the following
lines to your config.toml:

```toml
[imports.google]
url = "https://raw.githubusercontent.com/google/rust-crate-audits/main/audits.toml"
```

## Aggregated projects

These audits are aggregated from the following Google projects:

- [ChromiumOS]
- [Fuchsia]

[ChromiumOS]: https://chromium.googlesource.com/chromiumos/third_party/rust_crates/+/refs/heads/main/cargo-vet/
[Fuchsia]: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/third_party/rust_crates/supply-chain/

## Disclaimer

As with the audits from its contributing projects, this aggregation is provided
on a best-effort basis. These audits should not be construed as reflecting
material safety or security properties of Rust crates. We do our best to
aggregate valuable information; use at your own risk.
