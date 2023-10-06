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

These audits are automatically aggregated from the following Google projects:

- [ChromiumOS]
- [Fuchsia]

and other [manual sources] from within Google.

[ChromiumOS]: https://chromium.googlesource.com/chromiumos/third_party/rust_crates/+/refs/heads/main/cargo-vet/
[Fuchsia]: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/third_party/rust_crates/supply-chain/
[manual sources]: https://github.com/google/rust-crate-audits/tree/main/manual-sources

## Disclaimer

As with the audits from its contributing projects, this aggregation is provided
on a best-effort basis. These audits should not be construed as reflecting
material safety or security properties of Rust crates. We do our best to
aggregate valuable information; use at your own risk.

## Auditing criteria

Google audits Rust crates using both built-in and custom cargo-vet criteria.
Below are the formal descriptions of the criteria used across Google. We
recommend cross-referencing these criteria with the corresponding
[auditing standards] for a better understanding of what they mean.

[auditing standards]: https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md

### Cryptography

#### `crypto-safe`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#crypto-safe)

> All crypto algorithms in this crate have been reviewed by a relevant expert.
>
> **Note**: If a crate does not implement crypto, use `does-not-implement-crypto`,
> which implies `crypto-safe`, but does not require expert review in order to
> audit for.

#### `does-not-implement-crypto`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#does-not-implement-crypto)

> Inspection reveals that the crate in question does not attempt to implement
> any cryptographic algorithms on its own.
>
> Note that certification of this does not require an expert on all forms of
> cryptography: it's expected for crates we import to be \"good enough\"
> citizens, so they'll at least be forthcoming if they try to implement
> something cryptographic. When in doubt, please ask an expert.

### Deployment

#### `safe-to-run`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#safe-to-run)

> This crate can be compiled, run, and tested on a local workstation or in
> controlled automation without surprising consequences, such as:
> * Reading or writing data from sensitive or unrelated parts of the filesystem.
> * Installing software or reconfiguring the device.
> * Connecting to untrusted network endpoints.
> * Misuse of system resources (e.g. cryptocurrency mining).

#### `safe-to-deploy`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#safe-to-deploy)

> This crate will not introduce a serious security vulnerability to production
> software exposed to untrusted input.
>
> Auditors are not required to perform a full logic review of the entire crate.
> Rather, they must review enough to fully reason about the behavior of all
> unsafe blocks and usage of powerful imports. For any reasonable usage of the
> crate in real-world software, an attacker must not be able to manipulate the
> runtime behavior of these sections in an exploitable or surprising way.
>
> Ideally, all unsafe code is fully sound, and ambient capabilities (e.g.
> filesystem access) are hardened against manipulation and consistent with the
> advertised behavior of the crate. However, some discretion is permitted. In
> such cases, the nature of the discretion should be recorded in the `notes`
> field of the audit record.
>
> For crates which generate deployed code (e.g. build dependencies or procedural
> macros), reasonable usage of the crate should output code which meets the
> above criteria.

### Soundness

#### `ub-risk-0`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#ub-risk-0)

> This crate cannot cause undefined behavior because it contains no unsafe code.

#### `ub-risk-1`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#ub-risk-1)

> A designated unsafe code reviewer has audited the unsafe code in this crate.
> No risk of causing undefined behavior was found.
>
> UB-RISK-1 crates are suitable for applications with the strictest memory
> safety requirements:
> - Safety documentation is comprehensive and precise. Unsafe APIs can be used
>   soundly.
> - Unsafe blocks rely on clear invariants and preconditions, and are
>   well-justified by them.
> - No way to cause undefined behavior was found during review.
>
> UB-RISK-1 crates are exceptionally well-documented and justified, leaving
> little to no room for error.

#### `ub-risk-1-thorough`

A more thorough version of `ub-risk-1`. See [thorough soundness audits] for a
description of "thorough" audits.

#### `ub-risk-2`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#ub-risk-2)

> A designated unsafe code reviewer has audited the unsafe code in this crate.
> It has been found to pose a trivial risk of causing undefined behavior.
>
> UB-RISK-2 crates are suitable for most applications:
> - Safety documentation is relatively comprehensive, though it may not be
>   adequately precise. Unsafe APIs can be used soundly with very minor caution.
> - Unsafe blocks may rely on informal invariants and preconditions. The
>   reasoning required to justify them may be especially difficult or
>   under-documented.
> - Undefined behavior may be possible under extraordinary circumstances.
>
> UB-RISK-2 crates are effectively "the average good crate". While they may
> have very slight (but real) soundness issues, they are safe to use in general
> without much worry. These crates may exhibit undefined behavior under
> "extraordinary circumstances", which is ultimately up to reviewer discretion.
> Users may expect that reasonable use of the crate will not cause undefined
> behavior.

#### `ub-risk-2-thorough`

A more thorough version of `ub-risk-2`. See [thorough soundness audits] for a
description of "thorough" audits.

[thorough soundness audits]: https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#thoroughness

#### `ub-risk-3`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#ub-risk-3)

> A designated unsafe code reviewer has audited the unsafe code in this crate.
> It has been found to pose a significant risk of causing undefined behavior.
>
> UB-RISK-3 crates are suitable for select applications:
> - Safety documentation may not be adequately comprehensive or precise. Unsafe
>   APIs can be used soundly with a decent amount of caution.
> - Unsafe blocks may rely on under-documented or inferred invariants and
>   preconditions. The reasoning required to justify them may rely on specific
>   interpretations of undefined behavior that are under-specified. Those
>   interpretations must not actively cause UB, and should be unlikely to begin
>   causing UB in the future.
> - Undefined behavior may be possible under uncommon circumstances.
>
> UB-RISK-3 crates may not uphold the typical standards required for unsafe
> code, but are still used because they have been widely adopted and will
> inevitably be leveraged by indirect dependencies. These crates may exhibit
> undefined behavior under \"uncommon circumstances\", which is ultimately up to
> reviewer discretion. A decent amount of experience with unsafe code will be
> required to avoid undefined behavior.

#### `ub-risk-4`

[Auditing standards](https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md#ub-risk-4)

> A designated unsafe code reviewer has audited the unsafe code in this crate.
> It has been found to pose a high risk of causing undefined behavior.
>
> UB-RISK-4 crates are unsuitable except in specific situations:
> - Safety documentation may be nonexistent. Unsafe APIs may be difficult to use
>   safely even with experience writing unsafe code and specific domain
>   expertise.
> - Unsafe blocks may rely on undocumented invarianats or platform-specific
>   behavior. It may be difficult or impossible to reason about all possible
>   situations that may cause undefined behavior. Even a best-effort review is
>   expected to miss at least some possible unsoundness.
> - Undefined behavior may be possible under common circumstances.
>
> UB-RISK-4 crates may have APIs that are difficult to use without causing
> undefined behavior. They may require a large amount of domain expertise to use
> correctly, have large unsafe APIs with insufficient documentation, or perform
> many operations from safe code that could cause undefined behavior.
