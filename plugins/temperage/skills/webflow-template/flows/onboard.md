# Flow: onboard

**Run this first**, once per Webflow site. The skill ships without a catalog
because a catalog names components, master pages, and folders that exist on
one site only. Onboarding runs on **every** surface, from a maintainer with a
working folder to someone on claude.ai with only the Webflow connector. The
measuring is identical; only **where the output is stored** differs (step 0).
"Configure for organization" also sets the store for everyone (steps 0 and 9).

**Output**, whichever store: one catalog entry per template family, the filled
conventions, the catalog index, the rulebook installed as a Webflow Agent
Instruction where the connector user's site role allows, the sha256 of each
thing written, and always the same content as **downloadable files** (step
9). A working folder also caches the site inventory (step 3).

**Writes.** Steps 1 to 7 only read, except page folders (step 6, confirmed).
The only Webflow writes are those folders and the **guidance mirror** under
the toolkit prefix (step 8, one confirmation per write). No record of any kind
goes to Agent Instructions, and nothing changes page content, styles,
components, or variables.

**Files.** Written in the layout of `../references/stores.md` section 2:
`sites/<shortName>/conventions.md`, `catalog/<family>.md`, `catalog/index.md`,
`sync-state.json`, and in a working folder the local site inventory (legacy
git layout: `../references/webflow-conventions.md`,
`../references/catalog/<family>.md`, `../references/sync-state.json`,
`../references/site-inventory.md`). Read: `../references/rules.md`,
`../references/catalog/README.md` (entry format), `../references/stores.md`
(adapters, `org.json`, the pointer block). A filled example for a fictional
site is in `../references/examples/`; never overwrite it. Lint:
`../scripts/catalog_lint.py` (step 7). Access check: `doctor.md` (step 0).
Mirror procedure: `sync.md` (step 8).

## 0. Choose the store

Decide this before promising anything, and say the answer. Two decisions, both
from `../references/stores.md`: the **source of truth** (guidance and
records), and whether the **Webflow mirror** is writable for this account.
Webflow is never the source of truth: it is the guidance mirror, written
whenever the site role allows, and it never holds a record.

| Source of truth | For | Where the catalog lives | What reviews a change |
| --- | --- | --- | --- |
| **Notion** | a team that uses Notion through a Claude connector | one child page per path under the configured parent page | page history and comments |
| **Working folder** | Cowork, Claude Code, Codex | `webflow-template/sites/<shortName>/` in a folder the user names, never inside the plugin | a pull request in a git checkout, Drive or OneDrive versions in a synced folder, nothing in a plain one |
| **Downloads only** | claude.ai with no document connector | files the user keeps and re-attaches | nobody |

Notion is the only page store; Confluence is not supported.

**Discovery.** Before asking anything, run `doctor.md` (the one
`search_instructions` probe, no filter) and the discovery order in `stores.md`
section 6:

1. **Webflow pointer.** Probe 200 and `rules/<prefix>.md` opens with a
   `webflow-template` pointer block: adopt it, say so, do not ask (a
   re-onboard, or a second site in a configured workspace).
2. **Administrator configuration.** The table in `SKILL.md` (Claude Code,
   Cowork) or `references/org.json` with a non-empty `sourceOfTruth`. If it
   names a store the pointer does not, show both and ask: the pointer is the
   site's answer, the configuration the organization's constraint
   (`allowedStores`).
3. **The working folder's cached `org.json`** from an earlier run.
4. **Ask once.** Notion connector in the tool list: confirm it in one
   sentence. None, on Cowork or Claude Code: ask for a working folder ("an
   absolute folder outside the plugin; a git checkout if you want a pull
   request"). None, on claude.ai: downloads only, and say what that means.
   Never ask non-technical submitters about git.

**Configure for organization.** The answer from step 4 becomes the
organization's: fill every `org.json` value (`sourceOfTruth`, `location`,
`instructionPrefix`, `allowedStores`, `sites`, `testedMcpVersion` from
`webflow_guide_tool`, `configuredBy`, `configuredOn`) and emit the
organization zip in step 9.

Record the answer in the working folder's `org.json` (if there is one), in the
conventions' "Toolkit settings" (Source of truth, Store location, Webflow
mirror), and, when the role allows, in the pointer block written in step 8.

**The probe answers once, for the whole conversation:**

- **200**: the mirror is readable; keep the result for step 2. Whether it is
  writable follows from the site role named in step 1 (Designer and Site
  manager write; Marketer and Content editor read only); the first write in
  step 8 is the measurement.
- **HTTP 403 `forbidden`** ("you cannot read this SiteAgentInstructions"): a
  Webflow **site role** gate (Reviewer, or an Enterprise custom role), per the
  access and entitlement table in `../references/unsupported.md`. It is not an
  OAuth scope, and no admin can fix it by granting one. Quote the error and
  the row's exact ask: a Webflow workspace admin assigns the built-in Designer
  or Site manager role (Site settings > Site access), or someone who has one
  runs `sync.md` push later. Then **skip the mirror** and carry on: steps 1 to
  7 are still worth having, guidance goes to the source of truth and the
  bundle, and other agents on the site will not see it until someone with the
  role syncs. **Never retry a 403 in a loop.**
- Anything else: classify it by the same table, stop, and report. No blind
  retries.

**Size ceiling: 256 KB per instruction.** A full site capture is far larger
and **must never be written to the store**, mirror or source of truth. The
mirror holds only the filled conventions, the catalog entries, the index, and
the rulebook. The step 3 capture stays in the working folder or is not kept,
and is regenerated by re-running step 3. Measure every body before writing;
over about 200 KB, trim or split, never truncate silently (`stores.md`, size
rule).

**What a page store gives up.** In a git checkout a catalog change is a pull
request: a second person reads the diff before anything reaches the site.
**A page store or a plain folder removes that reviewer.** Nothing fully
replaces them; these limit the damage, and the report says so:

- an explicit **confirmation before each write** to the source of truth and
  the mirror (step 8), path and body shown: the only review that path has;
- proposed families stay `status: proposed` (`isDraft: true` in the mirror),
  a build that uses one says it is unconfirmed, and a maintainer promotes it
  through `flows/maintain.md`, "Maintaining in a page store";
- each entry keeps its own `## Decisions` and changelog sections; Notion page
  history holds the diff, a plain folder has none;
- the download bundle is the portable copy; on downloads only it is the only
  history.

## 1. Preflight

- `webflow_guide_tool` once.
- `data_sites_tool > list_sites` (summary). Ask which site; record `site_id`,
  display name, and the short name used in Designer URLs.
- Read `../references/webflow-conventions.md`, a **template**: rows marked
  MEASURE are this flow's worklist. Set the **instruction prefix** in "Toolkit
  settings" (default `page-templates`; change it only if another team owns
  that path on the site). Everything else reads the prefix from there.
- Ask the user's **Webflow site role** on this site (Site settings > Site
  access: Site manager, Designer, Marketer, Content editor, Reviewer, or a
  custom role by name) and record it in the "Agent Instructions" table. The
  MCP server has no whoami, and the remedy for every refused call depends on
  the role.
- Say what step 0 chose and what the user leaves with: Notion pages with a
  version history, files in a folder (a pull request only in a git checkout),
  or files to keep; plus, when the role allows, confirmed guidance writes into
  Webflow.

## 2. Read what is already there

Reuse the step 0 probe result; do not call `search_instructions` again.
`read_instruction` every rule and skill file it returned, and note what
another team owns under `rules/` or a skill folder. **Never overwrite
instructions the toolkit does not own.** An existing `rules/<prefix>.md` or
`<prefix>` skill means a re-onboard: read them and the current guidance in the
source of truth the pointer names, and in step 8 use the sync flow's conflict
handling instead of blind creates. Record the result (200 or 403, what
exists, who owns what, the pointer) in the "Agent Instructions" section.

## 3. Inventory (read-only)

The site inventory (`site-inventory.json` and `site-inventory.md` in the
working folder; `../references/site-inventory.json` and `.md` in the legacy
layout) is a **local cache**: gitignored, never committed, never shared
through a store (`stores.md` section 1). Cache everything in the JSON; render
the Markdown from it with a timestamp and the JSON's sha256 (site header,
Instructions, Pages and folders, Components, Styles, Variables, Fonts, Forms,
Site scripts, Locales, CMS collections, Page trees sampled, Inconsistencies).
A re-onboard replaces both.

**The capture never leaves this step**: not to the mirror, not to a page
store, not into the bundle, not committed. Without a working folder, hold it
in the conversation and re-run this step when it is needed again. An optional
committed index of it in a git working folder stays under 64 KB, lists
components by **name and group with no ids**, and keeps only the master page
and folder ids the catalog names.

Measure each item below into its section of the conventions, with the call it
came from and the date. A row not measured stays UNMEASURED; no later flow may
treat it as known.

- **Pages and folders.** `data_pages_tool > list_pages`; `get_page_metadata`
  for likely exemplars. Folders are not returned directly: infer them from
  `parentId` values that are not pages, and their paths from children's
  `publishedPath`. Record ids and child counts.
- **Components.** `data_component_tool > get_all_components` with
  `includeProps`, `includeVariants`, `includeInstanceCount`: the one place in
  the toolkit all three flags belong, because the step 7 entries are built
  from this census. Everywhere else components are looked up by name
  (`../references/rules.md` rule 10). The tool needs a `pageId` even for this
  site-wide read; pass any. Record `group` and `description`. **Count the
  components and the distinct names and record whether names are unique**
  ("Component names"): the catalog keys on names, so a duplicate must be
  renamed or disambiguated by group before a family can name it. Re-checked at
  every inventory refresh.
- **Styles.** Prefix-filtered queries through `data_style_tool`, **never a
  full dump**. Record the naming system with real examples (prefixes,
  separators, utility and combo classes) and whether it is a house system,
  Client-First, or other: probe `padding-global`, `text-size-*` and
  `heading-style-*` and record what came back either way.
- **Variables.** `data_variable_tool > get_variable_collections`, then
  `get_variables` per collection. Record modes, the variables a template would
  bind (colors, type, spacing), and which collection is current and which
  legacy.
- **Fonts** (`data_fonts_tool > list_fonts`), **forms** (`data_forms_tool >
  list_forms`), **site scripts** (`data_scripts_tool > get_site_scripts`),
  **locales** (`data_sites_tool > get_site`), **CMS collections** (names only,
  `data_cms_tool`), and **assets** (`data_assets_tool > list_assets`; the
  count feeds the rate-limit probes).
- **Tracking convention**, two answers into "Site-level tracking". First,
  **delivery**: how tracking reaches a page (site scripts per
  `get_site_scripts`, a named shell component, per-page embeds, or none); name
  a component by name and list it in the family's Shell components row too.
  Second, **link extras**: read the links on the site's own CTAs on exemplar
  pages (`data_element_tool > get_all_elements`, then the link values from
  `data_element_settings_tool`) and record the query parameters and custom
  attribute names they carry (the UTM keys, the attribute name, or `none`),
  with one real sibling CTA as the source. This is the convention a build
  **matches** (rule 14), not a scheme to apply: record only what you read.
  Click listeners and GTM triggers are invisible to this server; never infer
  them. Leave a row UNMEASURED rather than guess: build Phase 6 then warns and
  skips its tracking check.
- **Branching.** `data_pages_tool > list_branches`: `200` (available) or
  `403 not_enterprise_plan_site` (not available). When available, try the read
  path (`get_all_elements` on a branch page id from `list_branches`) and record
  whether it returned the branch tree; writes and `create_branch` stay
  UNMEASURED until the first branch-mode build, and build Phase 3 warns so.
  Record how many live branches exist and whether any show `has_conflicts`.
- **Rate limits and response sizes.** The measurement the whole pacing rule
  depends on, and a property of **this** site's size, not of the MCP server:
  every element, props, settings, and builder call prefetches the site's CMS
  collections and resolves assets for the nodes it returns, so cost scales
  with the counts above. Work the probe table in the conventions' "Rate limits
  and response sizes" one call at a time, never in parallel:
  `get_all_elements` at depth 1, 2, 3 and -1 on the busiest page; a
  type-filtered `query_elements`; two calls at decreasing spacings; a bulk
  read immediately followed by an element read; and whether slot children
  come back at depth 3 **with** element ids. Write the pacing rule this site
  needs and the capability table (component props, loose-section content,
  slot children: automated or manual). On a small site "no limit was reached
  and no pacing is needed" is a fair answer; on a large one loose and slot
  content may be manual only. Say which. Start from rule 10's defaults and
  relax only what you measured.
- **Designer availability.** `designer_tool > get_current_page`. On failure,
  run the Designer foreground procedure (`build.md` Phase 0 step 5). Record
  whether the Bridge App was reachable and, when it was, **the output of
  `get_all_breakpoints` (ids, names, min and max widths, which is base)**.
  Never assume a breakpoint set: build Phase 6 and the outline frames read it
  from here.
- **Access observed.** Per tool family, read and write separately, and in
  particular whether `bulk_update_pages_schema_markup` is allowed (a role that
  reads schema often cannot write it). For every refused call: HTTP status,
  error `code`, the message verbatim, and the access-table row it matched
  (`../references/unsupported.md`). The row carries the exact ask; never
  invent a permission name.

Also list inconsistent patterns for human review: duplicate classes, one-off
components, orphan variables, pages that fit no family. **Do not standardize
the site automatically.**

## 4. Propose families

Cluster pages by shared components and layout into **3 to 6 families**, each
with a name and slug, a master page candidate (or none), a section outline
drawn from the exemplars, and a recommended template model:

- master exists and body sections are components: **hybrid**;
- master exists and the body is mostly loose elements: **duplicate-master**
  (the `loose` convention in `../references/catalog/README.md`, if step 3
  found it applies here);
- no master, but section components exist: **component-recipe**.

## 5. User decisions

At most three questions per turn; record every answer:

1. Confirm the families (add, merge, drop, rename).
2. Template model per family (accept or override).
3. Master page per family (id and slug).
4. Allowed folders for new pages per family.
5. Slug and title conventions (pattern, casing, separators).
6. Default SEO title and description patterns and Open Graph image per family.
7. Default JSON-LD schema type per family. Read the masters with
   `query_pages_schema_markup` first and match house usage.
8. Isolation default per family (`draft-main` or `branch`), only if branching
   is available.
9. Who publishes, and the cadence. With no rule yet, the default: the person
   whose Webflow MCP connection ran the build, named in the run report.

## 6. Pre-create page folders

Confirm the list with the user, then `designer_tool > create_page_folder` for
every allowed folder that does not exist (needs the Designer open with the
Bridge App). Doing it now means Claude.ai builds never need the Designer for
folders. Record the folder ids in each family entry and in the conventions.

Designer unreachable (`build.md` Phase 0 step 5 is how to try): do not
improvise. Name the missing folders in each family entry and the report, and
say the first build into one will stop until a human creates it. This is the
only Webflow write in steps 1 to 7, and it is skippable.

## 7. Write catalog and conventions

Same content in every store; step 8 puts it where step 0 decided.

- **Catalog entries**, one per confirmed family, in the format of
  `../references/catalog/README.md`: `sites/<shortName>/catalog/<family>.md`
  in the source of truth (`../references/catalog/<family>.md` legacy), the
  same body mirrored to `<prefix>/catalog/<family>.md` when the role allows,
  and one entry in the bundle. Start at `0.1.0` with `status: proposed`; write
  `1.0.0` with `status: promoted` only when the maintainer took every step 5
  decision in this conversation, recorded under `## Decisions`. Component
  **names** exactly as step 3 returned them, never from memory and never ids
  (`catalog_lint.py` rejects an id in a component column); master page and
  folder ids do go in.
- **Conventions.** Fill every MEASURE row from step 3, each with the call and
  date, and append the probes to the reconciliation log. Anything unmeasured
  stays UNMEASURED and goes into the report's "Decisions needed".
- **Catalog index**, exactly as `sync.md` specifies for `<prefix>/SKILL.md`.
  Proposed families get their own table, marked unconfirmed.
- **Lint.** With code execution: write the entries to files and run
  `python3 scripts/catalog_lint.py <catalog dir> --sync-state <sync state>`
  (drop `--sync-state` when there is no such file); fix every error. Without:
  walk the checklist in `../references/catalog/README.md` by hand and say the
  linter did not run.

## 8. Write to the store

In this order: the source of truth, then the Webflow mirror, then nothing
else (`stores.md` section 2 for the layout, section 3 for the adapters).

### (a) Source of truth

Through the adapter step 0 chose, one page or file per path, verbatim body:

| Path (under `webflow-template/sites/<shortName>/`) | Content |
| --- | --- |
| `conventions.md` | the filled conventions file from step 7 |
| `catalog/index.md` | the catalog index from step 7 (the body `sync.md` specifies) |
| `catalog/<family>.md` | one per family, `status: proposed` unless every decision was taken |
| `sync-state.json` | `siteId`, `lastSync`, and one row per mirrored path once step (b) has run |

Plus `webflow-template/org.json` at the store root when it does not exist yet
(the pointer, the prefix, the tested MCP version). Per path: check the size,
show the path and body (the first twenty lines and the size for a long one),
get an explicit yes for that single write, write it, record the sha256. A page
store's `write` creates or updates the page, which makes a new version. In a
working folder write the files; in a git checkout the maintainer commits and
opens the pull request. Downloads only: nothing here, step 9 is the delivery.

### (b) Webflow mirror

Only when step 0 found the mirror readable and the role can write. The layout
is the one `sync.md` defines, so a later sync compares rather than creates:

| Path | Content | `kind` | Draft? |
| --- | --- | --- | --- |
| `rules/<prefix>.md` | the **pointer block** (`stores.md` section 5) followed by the rulebook from `../references/rules.md`, verbatim | `rule` | no |
| `<prefix>/SKILL.md` | the catalog index from step 7 | `skill` | no |
| `<prefix>/conventions.md` | the filled conventions file from step 7 | `skill` | no |
| `<prefix>/catalog/<family>.md` | one per family | `skill` | **yes while `status: proposed`**, no once promoted |

`<prefix>/conventions.md` is what a build without store access reads, so the
measured facts travel with the catalog. Nothing else is added, and nothing
outside this layout is ever touched. For each path:

1. **Check the size** against the 256 KB ceiling; trim or split anything near
   it, never truncate silently. The site capture never goes here.
2. **Show the path and the body and ask for an explicit yes before that
   single write.** Per path, never one yes for the batch: this is the entire
   review the mirror has.
3. `create_instruction` when nothing is there, `update_instruction` when
   something is. A body that is neither yours nor what a previous onboarding
   wrote is a **conflict**: show both, let the user choose, never resolve it
   silently (`sync.md`, "Push").
4. Record the sha256 of the raw body in `sync-state.json` in the source of
   truth (path, hash, ISO timestamp, `direction: push`).

A write returning **HTTP 403 `forbidden`** means the role can read Agent
Instructions but not manage them (Marketer or Content editor; the access
table has the exact ask). Stop mirroring, list the paths already written so a
later sync compares rather than creates, say so, and do not retry. The source
of truth is complete regardless; a Designer or Site manager finishes the
mirror with `sync.md` push.

### (c) Nothing else

No brief, manifest, snapshot, candidate, or sync state is ever written to
Agent Instructions, by this flow or a build; records live in the working
folder and the source of truth (`stores.md` section 1). Draft records an older
version left under the prefix are listed by a re-onboard and left for the
maintain flow's Housekeeping to move and delete.

## 9. Hand over the downloads

**Every store, always**, as one bundle:

- **`<prefix>-catalog-bundle.json`**: one JSON object in the bundle format of
  `../references/stores.md` section 3: `schema` (2), `prefix`, `siteId`,
  `shortName`, `generatedAt` (ISO 8601), `sourceOfTruth` and `location`, and
  as markdown `rulebook`, `conventions`, and `index`; `families` (each with
  `slug`, `status`, `version`, `body`); and the empty `runs` and `candidates`
  arrays that builds fill later.
- The same content as **one self-contained HTML page**, for reading.

**Never a zip for the bundle**: chat accepts HTML, JSON and plain text and
rejects zip archives. The site capture is never in it. Tell the user: "Keep
this file. If a build ever says it cannot read the catalog for this site,
attach it and the build will use it."

**Organization zip** (configure for organization only): a copy of the skill
folder with `references/org.json` filled from step 0 and nothing else changed,
zipped with `webflow-template/` as its root, for an owner to upload at
**Organization settings > Skills > + Add** (Team and Enterprise; on for
everyone, recipients cannot edit it, a re-upload updates everyone at next
use). It is a skill upload, not a chat attachment, so the zip is fine. On
claude.ai without code execution, hand over the filled `org.json` and the
README's one-line zip command instead. For Claude Code administrators also
print the same values as a `pluginConfigs["temperage"].options` block for
managed settings (`stores.md` section 4). Then have the operator run
`doctor.md` from a second account to verify.

## 10. Verify the store is not public (once per site)

Run it the first time the mirror is written on a site and record the result in
the "Agent Instructions" section; skip it later, it is a property of the site.
It needs a role that can write instructions; if step 8 skipped the mirror,
record "not run" for the first Designer or Site manager who syncs.

Known: Agent Instructions are gated by Webflow site role (Reviewer and custom
roles cannot even read them), reach authorized MCP clients as site metadata,
have no publish path of their own, and never appear in page content. Strong
evidence they are not public, but **not** a vendor statement (none was found),
so verify once:

1. `create_instruction` a throwaway at `<prefix>/exposure-check.md` holding
   one invented marker string that appears nowhere else.
2. Ask the **human** to publish the site at their normal cadence, or to
   confirm a publish has happened since step 1. The skill never publishes
   (rule 8) and never asks for a publish to be brought forward.
3. After that publish, confirm the marker appears nowhere in public output:
   view source on the home page and one interior page, the published
   `sitemap.xml` and `robots.txt`, and a site search. A browser is enough.
4. `delete_instruction` the throwaway path.
5. Record the answer with its date: "marker not found in public output after
   a publish on `<date>`"; or, if it *was* found, stop writing the mirror, say
   so loudly, and keep guidance in the source of truth only.

## 11. Report

In chat: the source of truth and why, and whether the mirror was written;
families with model and master; the branching answer; Designer availability
and breakpoints; the component-name uniqueness answer; the rate-limit findings
and pacing rule; folder ids created; the inconsistencies list; anything still
UNMEASURED; what was written, store by store and path by path; and the step
10 result if it ran. Then by store:

- **Notion**: the parent page and the pages written, and: "each write was
  confirmed in this conversation; the page history is the diff; the proposed
  families stay proposed until a maintainer confirms them."
- **Working folder**: the folder and files written; in a git checkout, a
  branch or pull request per the maintainer's instruction (never commit
  unless asked).
- **Downloads only**: the bundle, and that every build needs it attached until
  a store is configured.
- **Webflow mirror**, always: the paths written, or the access-table row the
  probe matched with its exact ask (for the site-role row: a built-in Designer
  or Site manager role from a workspace admin, with the README's admin request
  template), and that other agents on the site will not see the guidance until
  someone with the role runs `sync.md` push.
- **Configure for organization**: the organization zip and where it goes, the
  `pluginConfigs` values for Claude Code administrators, and the `doctor.md`
  check from a second account.
