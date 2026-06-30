# cc-kb: the remediation knowledge base

This is the style and idiom guidance the `clean-core-remediator` agent reads while it writes a replacement. Think of it as the reference for *how* to write a fix. It is never the source for *which* successor to use; that always comes from SAP through the fix-step cascade (`release-state lookup`, the released-API catalog, and ATC tags).

The agent reads this directory at runtime from `.abapctl/reference/cc-kb/`. It ships filled in, so there's nothing to download before you start.

## What ships here

- `clean-core-framework-classifications.md`: the Clean Core framework and technology classifications (the A/B/C/D reference).
- `clean-abap-rules.md`: the Clean ABAP rules (from SAP/styleguides, Apache 2.0).
- `modern-abap-transforms.md`: modern ABAP language elements (from SAP/styleguides, Apache 2.0).
- `abap-unit-testing.md`: the ABAP Unit cheat sheet (from SAP-samples/abap-cheat-sheets, Apache 2.0).
- `cloud-development.md`: the ABAP for Cloud Development cheat sheet (from SAP-samples/abap-cheat-sheets, Apache 2.0).

Every file that came from an upstream SAP repository keeps its `Source:` and `License:` header. For the full attribution, see [`THIRD-PARTY-NOTICES.md`](../../../THIRD-PARTY-NOTICES.md) at the repo root.

## Adding your own guidance

Drop more markdown files in here and the remediator picks them up on its next run. That's the place for your team's conventions, domain patterns, successor maps, or gotchas. The review step suggests amendments you can fold back in. If you want to freshen the bundled SAP references, re-download them from the upstream repositories named in their headers.