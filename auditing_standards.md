# Rust Crate Auditing Standards

## Why we need standards for auditing

When auditing third-party crates, we're reading a standard described by a few short paragraphs and judging whether some code satisfies them. That judgment adds a new fact to our shared understanding of third-party code. As a legal analogy, an auditing criteria description is like a _law_. The job of the auditor is to play judge and decide whether some code upholds or breaks the law. From this analogy, it's easy to understand how different auditors may interpret the same criteria differently. Some auditors may be more lenient than others, and real-world experience uniquely informs our decisions.

To ensure that our audits are usable throughout Google, we need to be confident that different auditors will come to the same conclusions about the same code. These auditing standards increase our confidence through clarifying remarks, case studies, and required processes. Continuing the legal analogy, these standards are like case law.

## Summary

Below are a list of all the auditing criteria and the requirements for someone to audit for them. If you're a contributor looking for criteria you can help audit, this table can help point you towards criteria you're qualified to audit for.

| **Criteria**                          | **Requires**                                      |
|---------------------------------------|---------------------------------------------------|
| [`crypto-safe`]                       | **Cryptography expertise** and **Rust expertise** |
| [`does-not-implement-crypto`]         | **Generalist SWE**                                |
| [`safe-to-run`]                       | **Generalist SWE**                                |
| [`safe-to-deploy`]                    | **Generalist SWE**                                |
| [`ub-risk-0`]                         | **Automation** or **Generalist SWE**              |
| [`ub-risk-1`], [`ub-risk-1-thorough`] | **Unsafe Rust expertise**                         |
| [`ub-risk-2`], [`ub-risk-2-thorough`] | **Unsafe Rust expertise**                         |
| [`ub-risk-3`]                         | **Unsafe Rust expertise**                         |
| [`ub-risk-4`]                         | **Unsafe Rust expertise**                         |

[`crypto-safe`]: #crypto-safe
[`does-not-implement-crypto`]: #does-not-implement-crypto
[`safe-to-run`]: #safe-to-run
[`safe-to-deploy`]: #safe-to-deploy
[`ub-risk-0`]: #ub-risk-0
[`ub-risk-1`]: #ub-risk-1
[`ub-risk-1-thorough`]: #ub-risk-1-thorough
[`ub-risk-2`]: #ub-risk-2
[`ub-risk-2-thorough`]: #ub-risk-2-thorough
[`ub-risk-3`]: #ub-risk-3
[`ub-risk-4`]: #ub-risk-4

## Common criteria

### Cryptography

#### `crypto-safe`

Requires **Cryptography expertise** and **Rust expertise** 

Crates with this criteria contain implementations of cryptographic algorithms which have been reviewed by an expert and deemed acceptable. Cryptography is always mission-critical. Even though we don't expect to catch every issue in review, a crate audited as crypto-safe is sufficient for use.

#### Guidelines

*   An expert must review the cryptographic algorithms and deem them acceptable. Generalist SWEs and others without adequate experience in cryptographic algorithms may not audit for this criteria.
*   It is not acceptable to just compare the written code against reference pseudocode or another accepted implementation.
*   It is recommended that a cryptography expert work with a Rust language expert to verify that the implementation works as intended.

### `does-not-implement-crypto`

Requires **Generalist SWE**

Crates with this criteria do not implement cryptographic algorithms.

#### Criteria guidelines

*   Generalist SWEs have the ability to determine whether a crate contains implementations of cryptographic algorithms.
*   Many crates use but do not implement cryptographic algorithms. The way they use those cryptographic algorithms may have material implications on the security or soundness of the algorithms. These issues may be raised while auditing deployment criteria, but are not relevant to this criteria. Those crates may still be audited as `does-not-implement-crypto`.

## Deployment

### `safe-to-run`

Requires **Generalist SWE**

This criteria is built-in to cargo vet and describes a crate which "can be compiled, run, and tested on a local workstation or in controlled automation without surprising consequences". It lists a few examples of what it considers "surprising consequences" which will be repeated here along with any additional guidelines specific to Google.

#### Criteria guidelines

*   Generalist SWEs have the ability to determine whether a crate is safe to run.
*   Crates must not do any of the following unless it is their express purpose and have been explicitly directed to do so by a developer or user:
    *   Read or write data from sensitive or unrelated parts of the filesystem
    *   Install software or reconfigure the device
    *   Connect to untrusted network endpoints
*   Crates must not do any of the following under any circumstances:
    *   Misuse system resources (e.g. cryptocurrency mining).

### `safe-to-deploy`

Requires **Generalist SWE**

This criteria is built-in to cargo vet and describes a crate which "will not introduce a serious security vulnerability to production software exposed to untrusted input". It clarifies some specific practices which will be repeated here along with any additional guidelines specific to Google.

#### Criteria guidelines

*   While this criteria does not require specific expertise, a generalist SWE must have familiarity with all auditing criteria and standards. Many organizations have additional requirements for code to be safe to deploy which may be relevant to the crate being reviewed.
*   Per the criteria description:
    *   Reviewers are not required to perform a full logic review of the entire crate. Rather, they must review enough to fully reason about the behavior of all unsafe blocks and usage of powerful imports. For any reasonable usage of the crate in real-world software, an attacker must not be able to manipulate the runtime behavior of these sections in an exploitable or surprising way.
    *   Ideally, all unsafe code is fully sound, and ambient capabilities (e.g. filesystem access) are hardened against manipulation and consistent with the advertised behavior of the crate. However, some discretion is permitted. In such cases, the nature of the discretion should be recorded in the `notes` field of the audit record.
    *   For crates which generate deployed code (e.g. build dependencies or procedural macros), reasonable usage of the crate should output code which meets the above criteria.
*   This criteria is not a proper soundness review. See the "Soundness" group for criteria pertaining to soundness.
*   This criteria does not satisfy a general requirement for deploying code. Google's requirements for deploying code may vary across organizations.

## Soundness

Most criteria in this group require training and experience in reviewing unsafe Rust code. Getting approval to do these unsafe reviews varies across organizations, but generally requires learning a wide variety of unsafe Rust topics and doing in-person shadowing.

### Thoroughness

The goal of a soundness review is to correctly classify the code as either "sound" or "unsound", then assign a risk level based on the code classified as "unsound". A perfect test would classify all of the sound code as "sound" and all of the unsound code as "unsound", with no classification errors. Soundness reviews are already very precise and technical work, and reaching such high levels of confidence may require an inordinate amount of effort. With this limitation in mind, review "thoroughness" is a way to increase the effort applied to a review and the confidence of its conclusion.

A "thorough" soundness review aims to increase the sensitivity of the review and correctly classify all unsound code. It should be considered the "gold standard" of reviewing; it should not be feasible to more accurately detect unsoundness than with a thorough review. Because thoroughness only focuses on reducing the false negative rate, thoroughness only matters when auditing for `ub-risk-1` and `ub-risk-2`. If the code is known to be riskier than `ub-risk-2`, then the thoroughness of the review is not consequential.

#### All soundness reviews

For all soundness reviews, an unsafe Rust reviewer must:

*   Look at each line of unsafe code
*   Reason about the unsafe Rust patterns found

It is acceptable if similar-looking unsafe blocks are skimmed over during a review. It is recommended (but not required) for the reviewer to document each unsafe block with a comment.

#### Thorough soundness reviews

For thorough soundness reviews, an unsafe Rust reviewer must additionally:

*   Explicitly justify the code in each unsafe block
    *   All unsafe operations must be identified and the safety conditions for each must be addressed.
    *   If the review is done in a group, then any nontrivial reasoning should be voiced for discussion.
*   Document the justification for each unsafe block with a comment
    *   For structurally identical unsafe blocks, it is acceptable for the reasoning to be “same as above” or "ditto".

Unless an unsafe Rust reviewer is very experienced, a group of two or more should perform thorough soundness reviews.

[thorough]: #thorough-soundness-reviews

### `ub-risk-0`

Also called: "No unsafe code"

Requires **Automation** or **Generalist SWE** 

Crates with this criteria do not contain unsafe Rust code.

#### Criteria guidelines

*   Because this criteria merely describes whether a crate contains unsafe Rust code, generalist SWEs may audit for it. Unless there is unambiguously no unsafe code, automation may not audit for this criteria. Even if automation audits for this criteria, a real person **must** sign off on the final audit. As a baseline, automation should reject any code containing the string `unsafe`. It may - but is not required to - allow **only** the following exceptions if robust and very well-tested:
    *   When not a whole-word textual match: e.g. `struct Unsafe`, `UnsafeCell`, `let not_unsafe`, etc
    *   Comments: e.g. `// This is not unsafe`
    *   Literals: e.g. `"hello unsafe"`
    *   Tests: i.e. code gated behind `#[cfg(test)]`. Note that unsafe blocks that are included in downstream binaries (e.g. `#[cfg(debug_assertions)]`) may not be ignored.
    *   In the future, unsafe code that is disabled via `cfg` may be ignored if the disabled features are recorded with the audit and `cargo vet` handles these exclusions programmatically. See [this issue](https://github.com/mozilla/cargo-vet/issues/380) for tracking.

### `ub-risk-1`

Also called: "No detectable unsoundness", "Excellent soundness"

Requires **Unsafe Rust expertise**

Crates with this criteria contain unsafe Rust code which is very high quality and poses near-zero risk of introducing undefined behavior. This risk level can be considered the `crypto-safe` of soundness, and these crates are suitable for the most demanding situations.

#### Criteria guidelines

*   Auditing for this criteria requires expertise with unsafe Rust. See the group documentation for details.
*   Per the criteria description:
    *   Unsafe blocks rely on clear invariants and preconditions, and are well-justified by them.
    *   No way to cause undefined behavior was found during review.
    *   Safety documentation is comprehensive and precise. Unsafe APIs can be used soundly.
*   The unsafe code in this crate must be sound even when held to the highest possible standard.
*   It may not be good enough for a crate to have sound unsafe code if that unsafe code is too difficult to confidently review.
*   Unsafe blocks should have safety comments. We prefer standard safety justifications of the form `// SAFETY: <justification>` but any comments which fulfill the same purpose are acceptable.
*   Every unsafe trait and function must have safety documentation clearly describing the preconditions and postconditions (if any) which are required to maintain memory safety.

### `ub-risk-1-thorough`

A more [thorough] version of [`ub-risk-1`].

### `ub-risk-2`

Also called: "Negligible unsoundness", "Average good crate"

Requires **Unsafe Rust expertise**

Crates with this criteria contain unsafe Rust code which is of good quality and pose a trivial risk of causing undefined behavior. The "average good crate" typically falls in this risk level. They are suitable for most applications.

#### Criteria guidelines

*   Auditing for this criteria requires expertise with unsafe Rust. See the group documentation for details.
*   Per the criteria description:
    *   Safety documentation is relatively comprehensive, though it may not be adequately precise. Unsafe APIs can be used soundly with very minor caution.
    *   Unsafe blocks may rely on informal invariants and preconditions. The reasoning required to justify them may be especially difficult or under-documented.
    *   Undefined behavior may be possible under extraordinary circumstances.
*   Most crates involving unsafe code belong here. Avoid putting crates in `ub-risk-1` unless they genuinely meet the stated criteria.
*   The definition of "extraordinary circumstances" is open to interpretation. Some examples of circumstances that can cause UB generally considered "extraordinary" are:
    *   Violating some obvious but unwritten rules about how to use an unsafe API. Even if an unsafe API doesn't specify that a pointer must be properly aligned, it's safe to assume that passing an unaligned pointer can cause UB.
    *   Violating work-in-progress rules around unsafe code that are being designed by `t-opsem` (e.g. Stacked Borrows and Tree Borrows), provided that stable alternatives for performing the same operation do not exist or have only recently been stabilized. This may include code that triggers errors in MIRI.
    *   Working in gray areas of unsafe semantics that are still under discussion and yet to be decided by `t-opsem`, provided that stable alternatives for performing the same operation do not exist or have only recently been stabilized, or that the general trend of current discussion of `t-opsem` can be shown to be in the direction that allows the pattern to be sound.
    *   Being able to cause UB with malicious code. The code should be complex enough that it would never be written by accident. For example: panicking in a callback you gave to the API, then catching it and performing some specific operations that normal code would not do.
    *   Using or implementing `#[doc(hidden)]` items to cause UB without unsafe code.
*   Users may expect that reasonable use of these crates will not cause UB.

### `ub-risk-2-thorough`

A more [thorough] version of [`ub-risk-2`].

### `ub-risk-3`

Also called: "Mild unsoundness", "Suboptimal soundness"

Requires **Unsafe Rust expertise**

Crates with this criteria contain unsafe Rust code which doesn't uphold the typical standards required for unsafe code. They MAY pose a risk of causing undefined behavior, and probably should be allowed on a case-by-case basis based on usage.

#### Criteria guidelines

*   Auditing for this criteria requires expertise with unsafe Rust. See the group documentation for details.
*   Per the criteria description:
    *   Safety documentation may not be adequately comprehensive or precise. Unsafe APIs can be used soundly with a decent amount of caution.
    *   Unsafe blocks may rely on under-documented or inferred invariants and preconditions. The reasoning required to justify them may rely on specific interpretations of undefined behavior that are under-specified. Those interpretations must not actively cause UB, and should be unlikely to begin causing UB in the future.
    *   Undefined behavior may be possible under uncommon circumstances.
*   These are crates that we would prefer not to use because of their unsafe code, but we may still do so begrudgingly.
*   The definition of "uncommon circumstances" is open to interpretation. Some examples of circumstances that can cause UB generally considered "uncommon" are:
    *   Leveraging incorrect variance on type lifetimes to violate memory safety.
    *   Writing implementations of traits not marked `unsafe` by violating documented invariants.
    *   Writing implementations of traits not marked `unsafe` that are not really intended to be implemented by the user.
    *   Explicitly forgetting values that have important drop behavior to cause UB when combined with operations that would not be expected to follow normally.
*   Some caution may be required to avoid undefined behavior.


`ub-risk-3` is expected to live at the threshold of acceptance as a dependency for most codebases: there is unsoundness in the crate, but there are also many situations where one need not worry about that unsoundness ever being triggered, and the codebase needs to make a judgement call. For example, one may choose to depend on a `ub-risk-3` dep if the rating is due to:

 * Unsoundness that is only triggered on platforms that the codebase does not care about.
 * Unsoundness that may become a problem in future versions of Rust, where the codebase is confident of having upstream patches by then.
 * Unsoundness that is triggered by a particular pattern of use of an API, which the codebase does not care to do and is confident of being able to avoid long term.


This is true for all audit levels, but `ub-risk-3` audits in particular should strive to contain audit comments detailing the safety issues found, to aid in such judgement calls.

### `ub-risk-4`

Also called: "Extreme unsoundness", "Very risky"

Requires **Unsafe Rust expertise**

Crates with this criteria contain very dangerous unsafe rust code. They pose a risk of causing undefined behavior with typical use.

#### Criteria guidelines

*   Auditing for this criteria requires expertise with unsafe Rust. See the group documentation for details.
*   Per the criteria description:
    *   Safety documentation may be nonexistent. Unsafe APIs may be difficult to use safely even with experience writing unsafe code and specific domain expertise.
    *   Unsafe blocks may rely on undocumented invariants or platform-specific behavior. It may be difficult or impossible to reason about all possible situations that may cause undefined behavior. Even a best-effort review is expected to miss at least some possible unsoundness.
    *   Undefined behavior may be possible under common circumstances.
*   Most crates that try to be sound but don't quite make the cut go in `ub-risk-3`, not here. These crates are wildly unsafe, and the only time we should use them is when they are a necessary evil.
*   Everything worse than `ub-risk-3` goes in here and we should do our best to avoid using them.

## Criteria-agnostic guidelines

### Delta audits should describe the final version

In general, delta audits should always take the previous, baseline audit into
account.  In particular, note that
[Cargo Vet's documentation](https://mozilla.github.io/cargo-vet/audit-entries.html#delta)
asks the auditor to acquire sufficient context when working on delta
audits:

> The [auditing] standard here is that the properties are actually
> preserved, not merely that that the diff doesn't obviously violate
> them. It is the responsibility of the auditor to acquire sufficient
> context to certify the former.

Since the delta audits only determine which criteria from the baseline version
are preserved, you should perform and record a non-delta audit to lower the
`UB-risk-N` level, or to downgrade from `crypto-safe` to
`does-not-contain-crypto`. In other words, a delta audit should never result in
lowering the `UB-risk-N` value from the previous audit, or downgrading
`crypto-safe` to `does-not-contain-crypto`.

For example, even though
[the 1.0.79 to 1.0.80 delta of the `proc-macro2` crate](https://chromium-review.googlesource.com/c/chromium/src/+/5453972/2..6)
doesn't add or change any `unsafe` code, it shouldn't be recorded as
`ub-risk-0`, because there is still `unsafe` outside of the delta
(e.g. `unsafe { imp::Literal::from_str_unchecked(repr) }` in
[`lib.rs` on line 1282](https://chromium-review.googlesource.com/c/chromium/src/+/5453972/2..6/third_party/rust/chromium_crates_io/vendor/proc-macro2-1.0.80/src/lib.rs#1282)).

Another example is the improvements in `flate2` between version 1.0.27
(`ub-risk-4` because of https://github.com/rust-lang/flate2-rs/issues/220) and
1.0.28 (`ub-risk-3` because of https://github.com/rust-lang/flate2-rs/pull/373).
Recording this improvement as a delta audit would be insufficient to pass `cargo
vet check` presumits (as tested by
[this ad-hoc Chromium CL](https://crrev.com/c/5532326)).

### Audits should be as precise as possible

*e.g. Audits should not account for `config.toml` policy*

Audits should record the most precise criteria for each crate, without
accounting for a project's `config.toml` policy. In some cases, a project's
policy may only require a crate to meet a minimum level of certification.
Although it may be less work to audit a crate at that minimum level, we
should always audit crates with the most precise criteria available. This
ensures that our audits are as shareable as possible between project
participants.

For example, Chromium requires `ub-risk-2` for crates used in the browser
process. The following `config.toml` is auto-generated for the quote crate:
entry for the `quote` crate:

```
# supply-chain/config.toml:
[policy."quote:1.0.36"]
criteria = ["does-not-implement-crypto", "safe-to-deploy", "ub-risk-2"]
```

This means that certifying this crate as `ub-risk-2` is sufficient for `cargo
vet` checks/presubmits to pass and `cargo vet check` will actually only ask
to certify at the `ub-risk-2` level:

```
$ tools/crates/run_cargo_vet.py check
Vetting Failed!

1 unvetted dependencies:
  quote:1.0.36 missing ["safe-to-deploy", "does-not-implement-crypto", "ub-risk-2"]

recommended audits for safe-to-deploy, does-not-implement-crypto, ub-risk-2:
    Command                             Publisher  Used By                                    Audit Size
    cargo vet diff quote 1.0.35 1.0.36  dtolnay    syn, chromium, prost-derive, and 6 others  6 files changed, 568 insertions(+), 548 deletions(-)

estimated audit backlog: 1116 lines

Use |cargo vet certify| to record the audits.
```

Although it is recommended to audit at `ub-risk-2`, the audit should always
certify `quote` 1.0.36 at the most precise ub-risk level possible. In this case,
the crate should be audited as `ub-risk-0` because it doesn't contain any
unsafe code.
