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

## Unsafe audit levels

Our audits attempt to be precise about the level of safety in a crate, since it can often be a sliding scale of trust even after thorough review. We have five "UB risk" levels used in the audits, documented here for convenience:


### UB-RISK-0

This crate cannot cause undefined behavior because it contains no unsafe code.

### UB-RISK-1

A designated unsafe code reviewer has audited the unsafe code in this crate. No
risk of causing undefined behavior was found.

UB-RISK-1 crates are suitable for applications with the strictest memory safety
requirements:
- Safety documentation is comprehensive and precise. Unsafe APIs can be used
  soundly.
- Unsafe blocks rely on clear invariants and preconditions, and are
  well-justified by them.
- No way to cause undefined behavior was found during review.

UB-RISK-1 crates are exceptionally well-documented and justified, leaving little
to no room for error.

### UB-RISK-2

A designated unsafe code reviewer has audited the unsafe code in this crate. It
has been found to pose a trivial risk of causing undefined behavior.

UB-RISK-2 crates are suitable for most applications:
- Safety documentation is relatively comprehensive, though it may not be
  adequately precise. Unsafe APIs can be used soundly with very minor caution.
- Unsafe blocks may rely on informal invariants and preconditions. The reasoning
  required to justify them may be especially difficult or under-documented.
- Undefined behavior may be possible under extraordinary circumstances.

UB-RISK-2 crates are effectively \"the average good crate\". While they may have
very slight (but real) soundness issues, they are safe to use in general without
much worry. These crates may exhibit undefined behavior under \"extraordinary
circumstances\", which is ultimately up to reviewer discretion. Users may expect
that reasonable use of the crate will not cause undefined behavior.


### UB-RISK-3

A designated unsafe code reviewer has audited the unsafe code in this crate. It
has been found to pose a significant risk of causing undefined behavior.

UB-RISK-3 crates are suitable for select applications:
- Safety documentation may not be adequately comprehensive or precise. Unsafe
  APIs can be used soundly with a decent amount of caution.
- Unsafe blocks may rely on under-documented or inferred invariants and
  preconditions. The reasoning required to justify them may rely on specific
  interpretations of undefined behavior that are under-specified. Those
  interpretations must not actively cause UB, and should be unlikely to begin
  causing UB in the future.
- Undefined behavior may be possible under uncommon circumstances.

UB-RISK-3 crates may not uphold the typical standards required for unsafe code,
but are still used because they have been widely adopted and will inevitably be
leveraged by indirect dependencies. These crates may exhibit undefined behavior
under \"uncommon circumstances\", which is ultimately up to reviewer discretion.
A decent amount of experience with unsafe code will be required to avoid
undefined behavior.

### UB-RISK-4

A designated unsafe code reviewer has audited the unsafe code in this crate. It
has been found to pose a high risk of causing undefined behavior.

UB-RISK-4 crates are unsuitable except in specific situations:
- Safety documentation may be nonexistent. Unsafe APIs may be difficult to use
  safely even with experience writing unsafe code and specific domain expertise.
- Unsafe blocks may rely on undocumented invarianats or platform-specific
  behavior. It may be difficult or impossible to reason about all possible
  situations that may cause undefined behavior. Even a best-effort review is
  expected to miss at least some possible unsoundness.
- Undefined behavior may be possible under common circumstances.

UB-RISK-4 crates may have APIs that are difficult to use without causing
undefined behavior. They may require a large amount of domain expertise to use
correctly, have large unsafe APIs with insufficient documentation, or perform
many operations from safe code that could cause undefined behavior.
