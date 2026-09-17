# Flow: onboard

**Run this first.** The skill ships without a catalog, because a catalog names
components, master pages, and folders that exist on one site only. Onboarding
is what turns the skill into a skill for *your* site.

Run once per Webflow site. It runs on **every** surface, not only Claude Code:
a maintainer with a working folder, or a non-technical person on claude.ai with
nothing but the Webflow connector and this skill. The measuring is identical.
What differs is **where the output is stored**, and that is step 0. An
administrator also runs it once per organization, with "configure for
organization", to set the store for everyone (step 0 and step 9).

Output, whichever store is used: one catalog entry per template family, a
filled `webflow-conventions.md`, a catalog index, the rulebook installed as a
Webflow Agent Instruction wherever the connector user's site role allows it,
and the sha256 of each thing written. A working folder also keeps a cached
site inventory; elsewhere the capture is not kept at all (step 3). And,
always, the same content as **downloadable files** (step 9).

Steps 1 to 7 are the same on every surface; they only read the site, except
page folders in step 6, which are created with confirmation. Only steps 0, 8, 9
and 11 depend on the store.

References written by this flow, in the layout `../references/stores.md`
section 2 defines: `sites/<shortName>/conventions.md`, `catalog/<family>.md`,
`catalog/index.md`, `sync-state.json`, and, in a working folder only, the local
site inventory (in the legacy git-checkout layout these are
`../references/webflow-conventions.md`, `../references/catalog/<family>.md`,
`../references/sync-state.json`, `../references/site-inventory.md`). References
read: `../references/rules.md`, `../references/catalog/README.md` (entry
format), `../references/stores.md` (adapters, `org.json`, the pointer block).
A filled example of what this flow produces is in `../references/examples/`;
it is a fictional site and this flow never overwrites it.

Scripts: `../scripts/catalog_lint.py` (step 7). Access check: `doctor.md`
(step 0). Mirror procedure: `sync.md` (step 8).

Nothing in this flow changes page content, styles, components, or variables.
The only Webflow writes are page folders (step 6, with confirmation) and the
**guidance mirror** under the toolkit prefix (step 8, with a confirmation
before each one). No record of any kind is written to Agent Instructions.

## 0. Choose the store

Decide this before promising anything, and say the answer out loud. Two
decisions, both from `../references/stores.md`: the **source of truth**, where
guidance and records live; and whether the **Webflow mirror** is writable for
this account. Webflow is not a source-of-truth option: it is the guidance
mirror in every configuration, written whenever the connector user's site role
allows, and it never holds a record.

| Source of truth | For | Where the catalog lives | What reviews a change |
| --- | --- | --- | --- |
| **Confluence** | a team that already uses Confluence through a Claude connector | one page per path under the configured parent page | page versions and comments |
| **Notion** | a team that already uses Notion through a Claude connector | one child page per path under the configured parent page, the file attached | page history and comments |
| **Working folder** | Cowork, Claude Code, Codex | files under `webflow-template/sites/<shortName>/` in a folder the user names, never inside the plugin; a git checkout gets a pull request | the folder's own history: a pull request in a git checkout, Drive or OneDrive versions in a synced folder, nothing in a plain one |
| **Downloads only** | claude.ai with no document connector | files the user keeps and re-attaches | nobody; the user keeps the files |

Confluence and Notion are peers (connector neutrality): name both, in this
order, and never call either the default.

### Run the discovery order first

Before asking anything, run `doctor.md` (it makes the `search_instructions`
probe, exactly once, with no filter) and the discovery order in `stores.md`
section 6:

1. **Webflow pointer.** If the probe returned 200 and `rules/<prefix>.md`
   opens with a `webflow-template` pointer block, the store is decided: adopt
   it, say so, and do not ask. This is a re-onboard, or a second site in a
   workspace that is already configured.
2. **Administrator configuration.** `${user_config.source_of_truth}` (Claude
   Code, Cowork) or `references/org.json` in the skill copy with a non-empty
   `sourceOfTruth`. Use it. If it names a store the pointer does not, show
   both and ask which is right: the pointer is the site's answer and the
   configuration is the organization's constraint (`allowedStores`).
3. **The working folder's cached `org.json`** from an earlier run on this
   machine.
4. **Ask once.** Detect which document connectors are present in the
   conversation's tool list. Two connected: "Confluence and Notion are both
   connected. Which one is the source of record for the Webflow template
   catalog?" One connected: confirm it in one sentence. None, on Cowork or
   Claude Code: ask for a working folder path ("an absolute folder outside the
   plugin; a git checkout if you want a pull request"). None, on claude.ai:
   downloads only, and say what that means. Non-technical submitters are not
   asked about git at all.

**Configure for organization.** When the operator says they are setting the
store up for the whole organization, take the answer from step 4 as the
organization's, fill every value of `org.json` (`sourceOfTruth`, `location`,
`instructionPrefix`, `allowedStores`, `sites`, `testedMcpVersion` from the
`webflow_guide_tool` response, `configuredBy`, `configuredOn`), and remember
to emit the organization zip in step 9.

Record the answer: in the working folder's `org.json` when there is a working
folder, in the conventions file's "Toolkit settings" (Source of truth, Store
location, Webflow mirror), and, when the role allows, in the pointer block
written in step 8.

### The probe result, exactly once

The `search_instructions` probe in `doctor.md` runs once and its answer holds
for the conversation:

- **200** — the mirror is readable. Keep the result for step 2. Whether it is
  also writable follows from the site role the user names in step 1 (Designer
  and Site manager write; Marketer and Content editor read only); the first
  write in step 8 is the measurement.
- **HTTP 403 `forbidden`** ("you cannot read this SiteAgentInstructions") — a
  Webflow **site role** gate, classified by the access and entitlement table in
  `../references/unsupported.md`: the connector user's role cannot read Agent
  Instructions (Reviewer, or an Enterprise custom role). It is not an OAuth
  scope, and no admin can fix it by granting one. Say exactly that, quote the
  error, and repeat the row's exact ask: a Webflow workspace admin assigns the
  built-in Designer or Site manager role (Site settings > Site access), or a
  maintainer who already has one runs `sync.md` push later. Then **skip the
  mirror** and carry on with the source of truth: everything steps 1 to 7
  measure is still worth having, guidance goes to the source of truth and the
  bundle, and other agents on the site will not see it until someone with the
  role syncs. **Never retry a 403 in a loop.** One probe, one answer, one
  mirror decision that holds for the rest of the conversation.
- Any other error: classify it by the same table, stop and report it. Do not
  retry blindly.

### Size ceiling: 256 KB per instruction

Each Agent Instruction body is capped at **256 KB**. A full site capture is far
larger than that and **must never be written to the store**, mirror or source
of truth. Only four kinds of thing go to the mirror: the measured answers (the
filled conventions file), the catalog entries, the catalog index, and the
rulebook. The capture from step 3 stays in the working folder or is simply not
kept (page stores and downloads); it is regenerated by re-running step 3, which
is cheap compared with storing it. Before every write, measure the body;
anything over about 200 KB is trimmed or split, and never silently truncated
(`stores.md`, size rule).

### What a page store gives up, said plainly

In a git checkout, a catalog change is a pull request: a second person reads a
diff before anything reaches the site. **A page store or a plain folder removes
that reviewer.** Nothing fully replaces them; these limit the damage, and the
report says so:

- **an explicit confirmation before each write** of guidance to the source of
  truth and to the mirror (step 8), with the path and the body shown, which is
  the only review that path has;
- **candidates stay proposed.** Families this flow proposes are written with
  `status: proposed` (and `isDraft: true` in the mirror), and a build that uses
  one says the family is unconfirmed. A maintainer promotes it through
  `flows/maintain.md`, "Maintaining in a page store";
- **the entry carries its own history.** Each entry keeps its `## Decisions`
  and changelog sections; page versions (Confluence) or page history (Notion)
  hold the diff, and a plain folder has none;
- **the download bundle is the portable copy.** Keep every bundle the flow
  hands over; on downloads only it is the only history there is.

## 1. Preflight

- Call `webflow_guide_tool` once.
- `data_sites_tool > list_sites` (summary). Ask the user which site; record
  `site_id`, display name, and the short name used in Designer URLs.
- Read `../references/webflow-conventions.md`. It is a **template**: every row
  is a placeholder and the ones marked MEASURE are this flow's worklist. Decide
  the **instruction prefix** now (default `page-templates`; change it only if
  another team already owns that path on the site) and write it into the
  "Toolkit settings" table. Everything else in the toolkit reads the prefix
  from there.
- Ask the user their **Webflow site role** on this site (Site settings > Site
  access shows it: Site manager, Designer, Marketer, Content editor, Reviewer,
  or a custom role by name) and record it in the conventions file's "Agent
  Instructions" table. The MCP server has no whoami, and the remedy for every
  refused call depends on the role (`../references/unsupported.md`, access and
  entitlement table).
- Say which source of truth step 0 chose and whether the Webflow mirror will
  be written, and what that means for the end of the flow: pages with a
  version history (Confluence, Notion), files in a folder with a pull request
  only if it is a git checkout (working folder), or a set of files to keep
  (downloads only); plus, when the role allows, a set of confirmed guidance
  writes into Webflow.

## 2. Read what is already there

Use the **step 0 probe result**; do not call `search_instructions` again for
this. `read_instruction` for every rule and skill file it returned. Note
anything under `rules/` or a skill folder that another team owns. **Never
overwrite instructions the toolkit does not own.** If `rules/<prefix>.md` or
the `<prefix>` skill already exist, this is a re-onboard: read them (the
pointer block in `rules/<prefix>.md` was adopted in step 0), then read the
current guidance from the source of truth it names, and in step 8 use the sync
flow's conflict handling instead of blind creates. Record the result (200 or
403, what already exists, who owns what, the pointer if any) in the conventions
file's "Agent Instructions" section.

## 3. Inventory (read-only)

The site inventory (`site-inventory.json` and `site-inventory.md` in the
working folder; `../references/site-inventory.json` and `.md` in the legacy
layout) is a **local cache**: it maps the whole site, so it is gitignored,
never committed, and never shared through a store (`../references/stores.md`
section 1). Cache everything into the JSON (the hashed artifact) and render the
Markdown from it with a timestamp and the sha256 of the JSON in the header:
site header, Instructions, Pages and folders, Components, Styles, Variables,
Fonts, Forms, Site scripts, Locales, CMS collections, Page trees sampled,
Inconsistencies. A re-onboard replaces both files.

**The capture never leaves this step.** It is not written to the Webflow
mirror (it is far over the 256 KB ceiling, step 0), not to a page store (the
size rule in `stores.md`), not into the download bundle, and it is not
committed. Without a working folder there is nowhere to cache it at all: hold
it in the conversation, read the answers out of it, and regenerate it by
re-running this step when it is needed again. What survives is the measured
answers, the catalog, and the index.

What goes into the source of truth is the catalog, the conventions file, the
index, and `sync-state.json`. If you also want a small committed index of the
capture in a git working folder, keep it under 64 KB, list components by
**name and group with no ids**, and keep only the master page and folder ids
the catalog names; anyone cloning the checkout regenerates the full capture by
re-running this step.

Measure each of the following and write the answer into the matching section of
`webflow-conventions.md`, with the call it came from and the date. A row you did
not measure stays UNMEASURED; no later flow may treat it as known.

- **Pages and folders.** `data_pages_tool > list_pages`; `get_page_metadata`
  for pages that look like exemplars. Folders are not returned directly: infer
  them from `parentId` values that are not pages and their paths from
  children's `publishedPath`. Record ids and child counts.
- **Components.** `data_component_tool > get_all_components` with
  `includeProps`, `includeVariants`, `includeInstanceCount`. This is the one
  place in the toolkit where all three include flags belong: the census is the
  site's full component inventory, taken once, and the catalog entries written
  in step 7 are built from the props, variants and instance counts it returns.
  Everywhere else the flags stay off and components are looked up by name
  (`../references/rules.md` rule 10). The tool needs a `pageId` argument even
  for this site-wide read; pass any page id. Record `group` and `description`
  metadata; these carry the family convention later. **Count the components and
  the distinct names and record whether names are unique**: the catalog keys on
  names, so a duplicate name has to be resolved (rename one, or disambiguate
  the catalog row by group) before a family can name it. This goes in the
  "Component names" table and is re-checked at every inventory refresh.
- **Styles.** Filtered queries by naming prefix through `data_style_tool`;
  **never a full dump** (the tool warns it returns a lot of data). Record the
  naming system in use with real examples (prefixes, separators, utility
  classes, combo classes), and say whether it is a house system, Client-First,
  or something else: probe for `padding-global`, `text-size-*` and
  `heading-style-*` and record what came back either way.
- **Variables.** `data_variable_tool > get_variable_collections`, then
  `get_variables` per collection. Record modes and the variables a template
  would bind to (colors, type, spacing), and which collection is current and
  which is legacy.
- **Fonts** (`data_fonts_tool > list_fonts`), **forms** (`data_forms_tool >
  list_forms`), **site scripts** (`data_scripts_tool > get_site_scripts`),
  **locales** (`data_sites_tool > get_site`), **CMS collections** (names only,
  `data_cms_tool`) and **assets** (`data_assets_tool > list_assets`; you need
  the count for the next bullet).
- **Tracking convention.** Two answers, both into "Site-level tracking". First,
  **delivery**: what `get_site_scripts` returned, and how tracking actually
  reaches a page — site scripts, a named shell component, per-page embeds, or
  none. Name a component by name, and list it in the family's Shell components
  row as well. Second, **link extras**: read the links already on the site's
  own CTAs on the exemplar pages (`data_element_tool > get_all_elements`, then
  the link values from `data_element_settings_tool`) and record the query
  parameters and the custom attribute names they carry — the UTM keys, the
  attribute name, or `none`. Record one real sibling CTA the answer came from,
  so a later handoff can point at it. This is the convention a build **matches**
  (rule 14); it is not a scheme to apply, so record only what you read. Click
  listeners and GTM triggers are invisible to this server: do not infer them,
  and leave either row UNMEASURED rather than guessing — Phase 6 then warns and
  skips its tracking check instead of trusting a guess.
- **Branching.** `data_pages_tool > list_branches`. Record `200` (available) or
  `403 not_enterprise_plan_site` (not available) as the site-wide answer. When
  available, also try the read path with a branch page id (`get_all_elements`
  on a page from `list_branches`) and record whether it returned the branch
  tree; writes and `create_branch` stay UNMEASURED until the first branch-mode
  build, and Phase 3 of the build flow repeats that as a warning. Record how
  many live branches exist and whether any show `has_conflicts`.
- **Rate limits and response sizes.** This is the one measurement the whole
  pacing rule depends on, and it is a property of **your** site's size, not of
  the MCP server. Every element, props, settings, and builder call prefetches
  the site's CMS collections and resolves assets for the nodes it returns, so
  the cost per call scales with the collection and asset counts you just
  recorded. Work through the probe table in `webflow-conventions.md`, "Rate
  limits and response sizes", one call at a time and never in parallel:
  `get_all_elements` at depth 1, 2, 3 and -1 on the busiest page; a
  type-filtered `query_elements`; two calls at decreasing spacings; a bulk read
  immediately followed by an element read; and whether slot children come back
  at depth 3 **with** element ids. Then write two things into that section: the
  pacing rule your site needs, and the capability table saying whether
  component props, loose-section content, and slot children are automated or
  manual here. On a small site the honest answer may be "no limit was reached
  and no pacing is needed"; say so. On a large one it may be that loose-section
  and slot content cannot be written at all and must be handed to the
  publisher; say that instead. Start conservative (the defaults in rule 10) and
  relax only what you measured.
- **Designer availability.** `designer_tool > get_current_page`. If it fails,
  run the Designer foreground procedure (`build.md` Phase 0 step 5: `open` the
  Bridge App link from the error message, wait about 40 seconds, re-probe, up
  to three times; on Claude.ai hand the user the link). Record whether the
  Bridge App was reachable during capture and, when it was, **the output of
  `get_all_breakpoints` (ids, names, min and max widths, which is base)** in
  the conventions file. Do not assume a breakpoint set: sites differ, and the
  build flow's Phase 6 and the outline's frames read the list from there.
- **Access observed.** For each tool family, record read and write separately,
  and in particular whether `bulk_update_pages_schema_markup` (page schema
  write) is allowed; a role that can read schema often cannot write it. For
  every refused call record the HTTP status, the error `code`, and the message
  verbatim, and the access-table row it matched (`../references/unsupported.md`).
  The row carries the exact ask; do not invent a permission name to ask for.

Separately, list inconsistent patterns for human review: duplicate classes,
one-off components, orphan variables, pages that fit no family. **Do not
standardize the site automatically.**

## 4. Propose families

Cluster pages by shared components and layout. Propose **3 to 6 families**,
each with: a name and slug, a master page candidate (or none), a section
outline drawn from the exemplars, and a recommended template model:

- master exists and body sections are components → **hybrid**;
- master exists and body is mostly loose elements → **duplicate-master**;
- no master but section components exist → **component-recipe**.

The `loose` convention in `../references/catalog/README.md` exists for the
second case. Whether it applies to your site is something step 3 found out, not
something to assume.

## 5. User decisions

Ask, at most three questions per turn, and record every answer:

1. Confirm the families (add, merge, drop, rename).
2. Template model per family (accept or override the recommendation).
3. Master page per family (id and slug).
4. Allowed folders for new pages per family.
5. Slug and title conventions (pattern, casing, separators).
6. Default SEO title and description patterns and Open Graph image per family.
7. Default JSON-LD schema type per family. Read the masters with
   `query_pages_schema_markup` first and match house usage rather than
   inventing one.
8. Isolation default per family, only if step 3 recorded branching as
   available: `draft-main` or `branch`.
9. Who publishes (names or role), and the publish cadence. If the team has no
   rule yet, the workable default is: the responsible publisher is the person
   whose Webflow MCP connection ran the build, named in the run report.

## 6. Pre-create page folders

For every allowed folder that does not exist yet: `designer_tool >
create_page_folder`. This needs the Designer open with the Bridge App. Doing it
now means Claude.ai build runs never depend on the Designer for folders.
Confirm the folder list with the user before creating; record the resulting
folder IDs in each family entry and in `webflow-conventions.md`.

When the Designer is not reachable (`build.md` Phase 0 step 5 is the procedure
for trying), do not improvise: record the missing folders as a manual item in
the report, name them in each family entry, and say that the first build into
one of them will stop until a human creates it in the Designer. Folder creation
is the only Webflow write in steps 1 to 7, and it is skippable.

## 7. Write catalog and conventions

The content is identical in every store; only where it lands differs. Write it
once, then step 8 puts it where step 0 decided.

- One catalog entry per confirmed family, in the format from
  `../references/catalog/README.md`: front matter plus the required headings.
  It is `sites/<shortName>/catalog/<family>.md` in the source of truth
  (`../references/catalog/<family>.md` in the legacy layout), the same body
  mirrored to `<prefix>/catalog/<family>.md` when the role allows, and one
  entry in the bundle. Entries start at version
  `0.1.0` with `status: proposed`; write `1.0.0` with `status: promoted` only
  when the maintainer took every step 5 decision in this conversation (the same
  rule as the maintain flow's create-family path), recorded in the entry's
  `## Decisions` section. Component **names** come from step 3, exactly as the
  component tool returned them, never from memory and never as ids;
  `catalog_lint.py` rejects an id in a component column. Master page ids and
  folder ids do go in the entry.
- Fill every MEASURE row of `../references/webflow-conventions.md` from step 3,
  each with the call and the date, and append the probes to the reconciliation
  log. Anything you could not measure stays UNMEASURED and is listed in the
  report's "Decisions needed".
- Build the **catalog index** exactly as `sync.md` specifies for
  `<prefix>/SKILL.md`, from the entries you just wrote. Proposed families are
  listed in their own table, marked unconfirmed, so a build can see them and
  say so.
- Lint. Working folder, or claude.ai with code execution enabled: write the
  entries to files and run `python3 scripts/catalog_lint.py <catalog dir>
  --sync-state <sync state>` (drop `--sync-state` when there is no such file).
  Fix every error before continuing. Without code execution, walk the checklist
  in `../references/catalog/README.md` by hand and say in the report that the
  linter did not run.

## 8. Write to the store

Three parts, in this order: the source of truth, then the Webflow mirror, and
nothing else. The layout is `../references/stores.md` section 2; the adapter
operations are section 3.

### (a) Source of truth

Through the adapter step 0 chose, one page or file per path, verbatim body:

| Path (under `webflow-template/sites/<shortName>/`) | Content |
| --- | --- |
| `conventions.md` | the filled conventions file from step 7 |
| `catalog/index.md` | the catalog index from step 7 (the body `sync.md` specifies) |
| `catalog/<family>.md` | one per family, `status: proposed` unless every decision was taken |
| `sync-state.json` | `siteId`, `lastSync`, and one row per mirrored path once step (b) has run |

Plus `webflow-template/org.json` at the store root when it does not exist yet
(the pointer, the prefix, the tested MCP version). For each path: check the
size (the rule in `stores.md`), show the user the path and the body (the first
twenty lines and the size is enough for a long one), ask for an explicit yes
before that single write, write it, and record the sha256. On a page store,
`write` creates the page when absent and updates it otherwise, which makes a
new version. On a working folder, write the files; in a git checkout, leave
the commit and the pull request to the maintainer. On downloads only, there is
nothing to write here: step 9 is the whole delivery.

### (b) Webflow mirror

Only when step 0 found the mirror readable and the role can write. The path
layout is the one `sync.md` defines, so a later sync compares rather than
creates:

| Path | Content | `kind` | Draft? |
| --- | --- | --- | --- |
| `rules/<prefix>.md` | the **pointer block** (`stores.md` section 5) followed by the rulebook from `../references/rules.md`, verbatim | `rule` | no |
| `<prefix>/SKILL.md` | the catalog index from step 7 | `skill` | no |
| `<prefix>/conventions.md` | the filled conventions file from step 7 | `skill` | no |
| `<prefix>/catalog/<family>.md` | one per family | `skill` | **yes while `status: proposed`**, no once promoted |

`<prefix>/conventions.md` is what a build with no store access reads instead
of the source of truth's copy; it exists so that the measured facts travel with
the catalog. Nothing else is added to the layout, and nothing outside it is
ever touched.

Then, for each path in the table:

1. **Check the size** of the body against the 256 KB ceiling. Trim or split
   anything near it; never truncate silently. The site capture is not written
   here at all.
2. **Show the user the path and the body** and **ask for an explicit yes
   before that single write**. Ask per path, not once for the batch: this
   confirmation is the entire review the mirror has, and a batch yes is not
   one.
3. Write it: `create_instruction` when nothing is there, `update_instruction`
   when something is. If the path already exists and its body is neither what
   you are writing nor what a previous onboarding wrote, that is a
   **conflict**: show both, let the user choose, and never resolve it silently
   (`sync.md`, "Push").
4. Record the sha256 of the raw body you wrote in `sync-state.json` in the
   source of truth (path, hash, ISO timestamp, `direction: push`), so a later
   sync or re-onboard can tell an unchanged path from an edited one.

If any write returns **HTTP 403 `forbidden`**, the connector user's site role
can read Agent Instructions but not manage them (Marketer or Content editor;
the access table in `../references/unsupported.md` has the exact ask). Stop
mirroring, list the paths that were written so a later sync compares rather
than creates, say so, and do not retry. The source of truth is complete
regardless; a Designer or Site manager finishes the mirror with `sync.md`
push.

### (c) Nothing else

No brief, manifest, snapshot, candidate, or sync state is ever written to
Agent Instructions, by this flow or by a build. Those are records; they live
in the working folder and the source of truth (`stores.md` section 1). A
previous version of this skill wrote candidates and run records into the
prefix as drafts: a re-onboard that finds such paths lists them and leaves them
for the maintain flow's Housekeeping to move into the source of truth and
delete.

## 9. Hand over the downloads

**Every store, always.** Whatever was written to Webflow or committed, the user
also leaves with the content in their hands. One bundle where practical:

- **`<prefix>-catalog-bundle.json`** — one JSON object in the bundle format
  `../references/stores.md` section 3 defines: `schema` (2), `prefix`,
  `siteId`, `shortName`, `generatedAt` (ISO 8601), `sourceOfTruth` and
  `location`, `rulebook` (the rulebook as markdown), `conventions` (the filled
  conventions file as markdown), `index` (the catalog index as markdown),
  `families` (one entry each with `slug`, `status`, `version`, and `body`),
  and the empty `runs` and `candidates` arrays that builds fill later.
- The same content as **one self-contained HTML page** when the user would
  rather read it than store it.

**Never a zip for the bundle.** Chat accepts HTML, JSON and plain text and
rejects zip archives, which is also why the bundle is one file rather than a
folder. The site capture is not in the bundle: it is large, it is regenerated
by re-running step 3, and the 256 KB reasoning in step 0 applies to
attachments too, for sanity if not for a hard limit.

Say what the bundle is for, in one line: "Keep this file. If a build ever says
it cannot read the catalog for this site, attach it and the build will use it."

**The organization zip** (only on "configure for organization"). Emit a copy
of the skill folder with `references/org.json` filled in from step 0 and
nothing else changed, zipped with `webflow-template/` as its root, for an
owner to upload at **Organization settings > Skills > + Add** (Team and
Enterprise; it is enabled for everyone, recipients cannot edit it, and a
re-upload updates everyone at next use). This is a skill upload, not a chat
attachment, so the no-zip rule does not apply; on claude.ai without code
execution, hand over the filled `org.json` and the one-line zip command from
the README instead, and let the operator zip it. For Claude Code
administrators, also print the same values as a `pluginConfigs["temperage"]
.options` block for managed settings (`stores.md` section 4). Then tell the
operator to run `doctor.md` from a second account to verify.

## 10. Verify the store is not public (once per site)

Do this **the first time the Webflow mirror is written on a site**, and record
the result in the conventions file's "Agent Instructions" section. Skip it on
later runs; it is a property of the site, not of the run. It needs a role that
can write instructions; when the mirror was skipped in step 8, record "not run"
and leave it for the first Designer or Site manager who syncs.

What is already known: Agent Instructions are gated by Webflow site role
(Reviewer and custom roles cannot even read them), are delivered to authorized
MCP clients as site metadata, have no publish path of their own, and never
appear in page content. That is strong evidence they are not public. It is **not** a vendor
statement — none was found — so verify it once rather than asserting it:

1. `create_instruction` a throwaway instruction at `<prefix>/exposure-check.md`
   containing one unique marker string that appears nowhere else (invent a
   nonsense token and write it down).
2. Ask the **human** to publish the site at their normal cadence, or to
   confirm a publish has happened since step 1. This skill never publishes
   (rule 8) and does not ask for a publish to be brought forward.
3. After that publish, confirm the marker appears nowhere in public output:
   view source on the published home page and one published interior page,
   check the published `sitemap.xml` and `robots.txt`, and run a site search
   for the marker. Anyone can do this in a browser; no tooling is needed.
4. `delete_instruction` the throwaway path.
5. Record the answer with its date: "marker not found in public output after a
   publish on `<date>`" or, if it *was* found, stop writing the mirror, say so
   loudly, and leave guidance in the source of truth only.

## 11. Report

Onboarding report in chat: the source of truth chosen and why, and whether the
mirror was written; families with model and master page; the branching answer;
Designer availability and the breakpoint list; the component-name uniqueness
answer; the rate-limit findings and the resulting pacing rule; folder IDs
created; the inconsistencies list; anything still UNMEASURED; what was written,
store by store and path by path; and the exposure check result if step 10 ran.

Then, by store:

- **Confluence or Notion**: the parent page and the pages written under it,
  and the sentence about review: "each write was confirmed in this
  conversation; the page history is the diff; the proposed families stay
  proposed until a maintainer confirms them."
- **Working folder**: the folder and the files written; in a git checkout, a
  branch or a PR per the maintainer's instruction (do not commit unless asked).
- **Downloads only**: the bundle, and the fact that every build will need it
  attached until a store is configured.
- **Webflow mirror**, in every case: the instruction paths written, or the
  access-table row the probe matched with its exact ask (for the site-role
  row: a built-in Designer or Site manager role from a Webflow workspace admin,
  with the admin request template in the README) and the note that other
  agents on the site will not see the guidance until someone with the role
  runs `sync.md` push.
- **Configure for organization**: the organization zip and where it goes
  (Organization settings > Skills), the `pluginConfigs` values for Claude Code
  administrators, and the instruction to run `doctor.md` from a second account
  to verify.
