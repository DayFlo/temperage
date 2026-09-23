# Run archive

Run records: the manifests, briefs, outlines, snapshots, and guard verdicts of
build runs. In the store layout (`../stores.md` section 2) this is
`sites/<shortName>/runs/` in the working folder and in the source of truth; in
a legacy git checkout it is this folder. A run record is a **record**, never
guidance: it is never written to a Webflow Agent Instruction (a version of this
skill before 1.1.0 did, under `<prefix>/runs/`; the maintain flow's
Housekeeping moves those into the source of truth). Nothing in this folder is
read by a build run except the open-manifest check.

This folder ships empty. It fills up as runs happen.

On Cowork, Claude Code, and Codex a live build writes its working files here
**first** (`<yyyy-mm-dd>-<slug>.brief.json`, `.manifest.json`,
`.outline.html`, `.pre.snapshot.json`, later `.post.snapshot.json` and
`.guard.verdict.json`) before any network store, so the record survives a 429,
a 403, or a closed tab; then the same files go to the source of truth through
its adapter. The build flow's Phase 0 step 4 checks this folder for an `open`
manifest with the same slug before starting a duplicate build.

What lands here:

- `<yyyy-mm-dd>-<slug>.manifest.json`: the run manifest
  (`../manifest-schema.md`), status `verified`, `failed`, or `cleaned`. Open
  runs are not archived; resume or clean them up first.
- `<yyyy-mm-dd>-<slug>.brief.json`: the approved brief the manifest's
  `briefHash` refers to.
- `<yyyy-mm-dd>-<slug>.md`: a run record a pre-1.1.0 version wrote to Webflow,
  moved here verbatim by the maintain flow's Housekeeping.

Why keep them: the `created.componentIds` lists are how the sync flow tells
run-created components (whose description it may rewrite) from the pre-existing
library (which it never touches), and the manifests are the audit trail for
what shipped at each publish.

Snapshots (`*.pre.snapshot.json`, `*.post.snapshot.json`) are bulk captures of
the site: gitignored in a checkout, kept in the working folder, written to a
page store only under the size rule in `../stores.md` (otherwise the manifest
carries their sha256 and location); regenerate them with
`scripts/build_snapshot.py`.

The worked example under `../../assets/examples/` is **not** a run: every step
in its manifest is `skipped`, `created` is empty, and it exists to show the
manifest shape. Never offer to resume or clean it up; the tests depend on it
staying as it is.
