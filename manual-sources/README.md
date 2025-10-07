# Manual audit sources

This repo aggregates audits from other sources like Android and Fuchsia; however it is the source of truth for some audits that are not otherwise public, for example for Rust crates being imported into the [google3](https://opensource.google/documentation/reference/glossary#google3) monorepo.

The primary source of truth for these is stored in audits.toml files in this folder, and aggregated into the toplevel audits.toml by CI.

If you are performing an audit for such a source, you may submit your audits to these files directly. You do not need to update the toplevel audits.toml as that will be done by CI, though you can if you'd like (it's finicky).

You may directly depend on these audit files if you wish, however their locations may move around in the long term. It is recommended to depend on the toplevel audit file in this repo.

`additional-audits.toml` contains additional audit criteria that are not assessed during private reviews.
For example, this file might include `safe-to-deploy` audits for crates that have been evaluated `ub-risk-2` or lower during unsafe review and do not perform potentially dangerous operations such as filesystem access.
