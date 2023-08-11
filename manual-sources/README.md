# Manual audit sources

This repo aggregates audits from other sources like Android and Fuchsia; however it is the source of truth for some audits that are not otherwise public, for example for Rust crates being imported into the [google3](https://opensource.google/documentation/reference/glossary#google3) monorepo.

The primary source of truth for these is stored in audits.toml files in this folder, and aggregated into the toplevel audits.toml by CI.

If you are performing an audit for such a source, you may submit your audits to these files directly. You do not need to update the toplevel audits.toml as that will be done by CI, though you can if you'd like (it's finicky).

These audit files are currently not useful on their own as they do not define the criteria (and implicitly use the ones from the toplevel audits.toml). Others should not depend on them.
