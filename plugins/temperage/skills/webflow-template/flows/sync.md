# Flow: sync

Maintainer flow. Keeps the **source of truth** and the **Webflow Agent
Instructions mirror** aligned, for guidance only. The source of truth
(`../references/stores.md`: Confluence, Notion, or a working folder) is where
rules, the index, the conventions, and catalog entries are reviewed; Webflow is
the mirror every agent on the site reads at runtime. After a sync the two
match. Records (briefs, manifests, snapshots, candidates, sync state) are not
part of this flow: they live in the source of truth and are never written to
Webflow.

References: `../references/rules.md`, `../references/stores.md` (adapters, the
pointer block), `../references/catalog/`, `../references/sync-state.json`
(the record this flow updates), `../references/webflow-conventions.md` (the
instruction prefix), `../references/unsupported.md` (the access and entitlement
table). Tools: `data_agent_instructions_tool > search_instructions`,
`read_instruction`, `create_instruction`, `update_instruction`,
`move_instruction`, `delete_instruction`, plus the source of truth's adapter
(`list`, `read`, `write`).

## The instruction prefix

Every path below is built from one setting: the **instruction prefix**, defined
once per site in `../references/webflow-conventions.md`, "Toolkit settings"
(the organization default is `instructionPrefix` in `org.json`). The default
is `page-templates`. Read it from there at the start of every sync and use it
everywhere; nothing else in the skill hard-codes it. If a site already has
another team's `page-templates` namespace, the maintainer changes the prefix in
the conventions file at onboarding and this flow follows.

Throughout this file, `<prefix>` means that value.

### The pointer block

`rules/<prefix>.md` opens with one fenced block that tells every agent on the
site where the source of truth is (`stores.md` section 5):

````
```webflow-template
sourceOfTruth: confluence
location: SPACE=<space key> PARENT=<page id>
instructionPrefix: page-templates
updated: <ISO date>
```
````

The push composes the body of `rules/<prefix>.md` as this block followed by
`references/rules.md` verbatim, and hashes the whole. A pull that finds the
block edited in Webflow treats it like any other guidance edit: show both
versions, never adopt a pointer silently, and refuse one that names a store
outside `allowedStores` in `org.json`. The block carries a location, never a
credential.

## Toolkit-owned paths

Sync touches these paths and **nothing else**. An instruction outside this list
is never read for sync purposes, never written, never deleted.

| Path | Content | Direction |
| --- | --- | --- |
| `rules/<prefix>.md` | The pointer block, then the rulebook from `references/rules.md` | push |
| `<prefix>/SKILL.md` | Catalog index and how to use it (spec below) | push |
| `<prefix>/conventions.md` | The filled conventions file, for a build that cannot reach the source of truth | push |
| `<prefix>/catalog/<family>.md` | **Promoted** family entries as non-drafts; a `proposed` entry at the same path as a **draft** (`isDraft: true`) until a maintainer promotes it | push |

Guidance edits made in the Webflow Instructions panel on any of the four paths
are **pulled** back into the source of truth (below). Nothing else is pulled:
candidates and run records are no longer written to Webflow, so there is
nothing of theirs to fetch. Paths of the shape `<prefix>/candidates/…` or
`<prefix>/runs/…` that a version before 1.1.0 wrote are listed by the maintain
flow's Housekeeping, moved into the source of truth, and deleted there, with
confirmation; sync leaves them alone.

Paths follow the skill-format convention (`<skill-name>/SKILL.md`,
`<skill-name>/<subfolder>/<file>.md`), each body up to 256 KB. That ceiling is
a hard constraint, not a guideline: a site capture is far larger and is never
written to the store (`flows/onboard.md` step 0). Only measured answers, the
conventions, the catalog entries, the index, and the rulebook go there.

`<prefix>/conventions.md` is the mirror's copy of `conventions.md`. It exists
because a build whose account cannot reach the source of truth still needs the
measured facts; push it whenever the source file changes, and read it before a
pull so an in-mirror edit is not silently lost.

**Sites onboarded before 1.1.0, or with the mirror as the only copy.** The
first sync from a freshly configured source of truth finds remote bodies it
never pushed and `sync-state.json` empty: that is the conflict case below, and
the answer is almost always "keep remote": pull the entries into the source of
truth as the starting point, then push from there afterwards. Never overwrite
an in-mirror catalog with an empty one.

## Step 0: probe the store

Before any write, `search_instructions` scoped to `<prefix>` (one call). Three
outcomes:

- **200 with hits or no hits**: continue. If `rules/<prefix>.md` is among the
  hits, read it and compare its pointer block with the source of truth this
  sync is running from; a difference is a conflict (below), never resolved
  silently.
- **HTTP 403 `forbidden`** ("you cannot read this SiteAgentInstructions"), on
  this call or on any later `create_instruction` or `update_instruction`:
  **stop the sync**. Classify it by the access and entitlement table in
  `unsupported.md` (it is a Webflow **site-role** gate: the connector user's
  role cannot read or manage Agent Instructions; it is not an OAuth scope),
  quote the error, repeat the row's exact ask, and leave `sync-state.json`
  exactly as it was (`siteId` and `lastSync` stay `null` if no sync ever
  happened). Nothing is half-pushed because the probe runs first; if a 403
  arrives mid-push after some paths were created, list the paths that were
  written so the next successful sync knows to compare rather than create.
- Any other error: classify it by the same table, stop and report; do not
  retry blindly.

Whatever the outcome, the probe runs **once**. A 403 is an answer, not a
transient failure; never retry it in a loop.

## Catalog index: `<prefix>/SKILL.md`

Generated by this flow at push time from the entries in the source of truth,
and written there as `catalog/index.md` too; never edited by hand in Webflow.
Content, in this order and nothing more:

1. Title `# <Site or team name> page templates` and one paragraph: this skill is
   the catalog of template families for the site; read `rules/<prefix>.md`
   first; every build reuses a family and never publishes.
2. A table with one row per **promoted** family: slug, version, template model,
   master page (id and leaf slug, or "none" for a recipe), allowed folders
   (paths), instruction path of the entry (`<prefix>/catalog/<slug>.md`). A
   second table below it lists the **proposed** families the same way, headed
   "proposed, unconfirmed by a maintainer", so a build can find them and say
   so; they are the draft entries described above.
3. How to use: read the entry for the family you pick; the entry **names** its
   components, so resolve those names against the live site before writing,
   with one `data_component_tool > query_components` filtered to the family's
   component group or batched `get_component` by `name`, plus
   `get_page_metadata` for the master page; a name that matches no component,
   or more than one, blocks the family, and so does a missing or moved master
   page; candidates and run records live in the source of truth named by the
   pointer block in `rules/<prefix>.md`, never here.
4. One footer line: `Generated by flows/sync.md from <source of truth> on
   <ISO date>; catalog lint ok, <n> families.`

The body is hashed like every other pushed path. A site with no promoted family
gets the title, the paragraph, an empty table, and the footer, so build runs
see that nothing is confirmed. A site that has not been onboarded has no index
at all, which is the same signal.

## sync-state.json

```json
{
  "siteId": "<site_id>",
  "lastSync": "<ISO 8601 timestamp>",
  "paths": {
    "rules/page-templates.md": {
      "hash": "sha256:<hash of the last-synced raw body>",
      "syncedAt": "<ISO 8601>",
      "direction": "push"
    },
    "page-templates/catalog/product-landing.md": {
      "hash": "sha256:...",
      "syncedAt": "...",
      "direction": "push"
    }
  }
}
```

The keys in that example use the default prefix; a site that changed the prefix
has its own keys. `siteId` and `lastSync` are `null` before the first sync, and
the file ships that way. The hash is the sha256 of the raw body as returned by
`read_instruction` with `resolve_references: false`, so it matches what was
actually stored. `catalog_lint.py --sync-state` checks that the file parses and
has these three keys. The file is a record: it lives in the source of truth
(`sites/<shortName>/sync-state.json`; `references/sync-state.json` in a legacy
git checkout) and is never mirrored.

## Push (source of truth → Webflow mirror)

Run `python3 scripts/catalog_lint.py <catalog dir> --sync-state <sync state>`
first (write the entries to files when the source of truth is a page store); a
failing lint stops the push. Then, for each of the rulebook (pointer block plus
`rules.md`), the generated catalog index, `conventions.md` (to
`<prefix>/conventions.md`), and every `catalog/<family>.md` (promoted as
non-draft, proposed as draft):

1. Compute `local` = sha256 of the source body (for the index: the body you
   are about to write).
2. `read_instruction` with `resolve_references: false` for the target path.
   Compute `remote` = sha256 of the returned body, or "absent".
3. Decide:
   - `remote` absent → `create_instruction` (`kind` `rule` for the rulebook,
     `skill` for the index, the conventions, and the entries; `isDraft` per
     the entry's status). Result: **created**.
   - `remote` == `local` → nothing. Result: **skipped**.
   - `remote` == last-synced hash from `sync-state.json` → safe to overwrite:
     `update_instruction`. Result: **updated**.
   - `remote` differs from both the last-synced hash and `local` → someone
     edited the instruction in Webflow since the last sync. **Conflict.** Show
     the diff (remote vs local) and let the user choose: keep remote (pull it
     into the source of truth as a proposed change), overwrite with local, or
     stop. Never resolve a conflict silently. `update_instruction` has no
     version check, so this hash comparison is the only protection.
4. Record the new hash and timestamp in `sync-state.json`, in the source of
   truth.

Every write is confirmed per path, with the path and the body shown, unless
the working folder is a git checkout whose pull request already reviewed the
change; then one confirmation for the batch is enough, and say which it was.

**Component description rewrite.** After pushing a family entry, rewrite the
`description` of each component the entry lists as `self`-owned **and that a
build run or this family created** (a promoted candidate, or a component created
through the create-family path) to `<family>@<version> | promoted`, confirming
once for the batch. Pre-existing library components, whether the entry marks
them `shared` or claims them as `self`, keep their Webflow `group` and
`description` untouched; the catalog entry is the record of ownership for those.
How to tell them apart: a run-created component's description already carries
the toolkit tag (`| candidate | run <slug>`), or its ID appears in
`created.componentIds` of a manifest under the source of truth's `runs/`.
`group` is never rewritten by sync.

## Pull (Webflow mirror → source of truth)

For guidance edits made in the Webflow Instructions panel:

1. `read_instruction` (`resolve_references: false`) on each of the four
   toolkit-owned paths that exist.
2. For each whose hash differs from the last-synced hash in `sync-state.json`:
   show the diff against the source of truth, and on the user's yes `write` it
   to the source of truth (`catalog/<family>.md`, `conventions.md`, or, for the
   rulebook and the index, note that they are generated: a rulebook edit
   becomes a proposed change to `references/rules.md`, an index edit is
   discarded and regenerated, and a pointer-block edit is adopted only after
   the user confirms it and `allowedStores` permits it). Record the hash in
   `sync-state.json` with `direction: pull`.
3. Never delete anything in this flow; pruning is the maintain flow's job and
   asks first.

## Summary

Sync ends with an updated `sync-state.json` and a summary table: path,
direction, result (created, updated, skipped, pulled, conflict), and the hashes
involved. Conflicts left unresolved are listed first. A sync stopped by a 403
ends with the access-table classification and its exact ask instead, and no
state change. In a git working folder, open a PR with the `sync-state.json`
change and any pulled guidance; on a page store, the updated pages are the
record.
