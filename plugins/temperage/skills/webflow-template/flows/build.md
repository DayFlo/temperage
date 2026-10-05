# Flow: build

The per-page workflow. In: a design and a submitter who can answer questions.
Out: an unpublished draft page built from a template family, a verification
report, and the brief and run manifest as records.

**Binding throughout:** `../references/rules.md`. By phase:
`../references/interview.md` (2), `../references/catalog/` (0, 3, 5),
`../references/brief-schema.md` and `../references/outline-spec.md` (4),
`../references/manifest-schema.md` (5 to 7), `../references/unsupported.md`
(1, 7), `../references/stores.md` (0, 5). Scripts:
`../scripts/validate_brief.py`, `../scripts/render_outline.py`,
`../scripts/diff_inventory.py`, `../scripts/manifest.py`. If a script cannot
run, follow its reference by hand and say so in the report.

Phases run in order. No Webflow write before Phase 5; no Phase 5 before the
outline is approved.

**Read the conventions before planning any phase**: `conventions.md` from the
source of truth (`webflow-conventions.md` in a legacy git checkout), else
`<prefix>/conventions.md` from the Webflow mirror, else the `conventions`
field of an attached bundle. Every site-specific fact a build depends on is
measured there at onboarding, never assumed here: mirror readability, pacing
(rule 10), whether loose-section and slot content is automated or manual,
whether the role can write page schema, branching, breakpoints, Designer
reachability, slugs, folders, naming, tracking. UNMEASURED means unknown: say
so in the report, never guess. `<prefix>` is the instruction prefix from
"Toolkit settings" (default `page-templates`).

## Phase 0: Preflight (silent unless something is wrong)

1. `webflow_guide_tool` once per conversation.
2. **Find the store and load the catalog.** Run the discovery order in
   `../references/stores.md` section 6 (skip what preflight already did), then
   its read order for guidance, stopping at the first that answers:
   - the **source of truth**: `catalog/index.md`, `conventions.md`, each
     `catalog/<family>.md`;
   - the **Webflow mirror** via `read_instruction`: `<prefix>/SKILL.md`,
     `<prefix>/conventions.md`, each `<prefix>/catalog/<family>.md`; say once
     "reading the Webflow mirror; it may lag the source of truth";
   - an **attached bundle**: say which `generatedAt` it carries; treat it as
     possibly stale;
   - else the bundled `../references/catalog/`, which ships empty: "This site
     has not been onboarded; I am using the copy bundled with the skill, which
     may be empty or stale."

   **403 `forbidden`** on `search_instructions` ("you cannot read this
   SiteAgentInstructions") is a Webflow **site role** gate, not an OAuth scope
   (`../references/unsupported.md`, access and entitlement table, has the
   exact ask). It does not stop the build: guidance comes from the source of
   truth or a bundle, `rules.md` still binds, and the mirror is neither read
   nor written this run. Say once: "The Webflow instruction mirror is not
   readable for this account's site role; I am reading the catalog from
   `<store>`." Classify any other non-200 by the same table.

   A source-of-truth entry with `status: proposed`, or a mirror entry that
   comes back as a **draft**, is a family onboarding wrote and no maintainer
   has confirmed. Usable; note it for Phases 3 and 7.

   **Third case: no store readable and no catalog bundle attached.** Never
   improvise a family. Say: "I cannot read a catalog for this site. Two ways
   forward: run `flows/onboard.md` against this site, or attach the catalog
   bundle a previous onboarding produced - the JSON or the HTML file;
   chat accepts HTML, JSON and plain text but not a zip." Use an attached
   bundle as above. With neither, state what is degraded before going on: the
   rulebook still applies, Phases 1 and 2 still work, and the run **stops at
   Phase 3**. Nothing site-specific is known (pacing, breakpoints, capability
   table, folder ids, slug and SEO conventions), so no Webflow write happens
   this run.
3. **Master pages**, only if scoring needs them: one batched
   `data_pages_tool > get_page_metadata` for the master of each family about
   to be offered (slug and parent folder must match the entry). No
   `get_all_elements` and no component list here: components are reconciled
   in Phase 3, for the chosen family only. A missing, moved, or renamed master
   **blocks that family** this run: say which and why, and point the
   maintainer at `flows/maintain.md` (Refresh inventory, then fix the entry).
   Other families stay usable.
4. **Open manifest for this slug** (re-check once the slug is known), in the
   read order for records (`stores.md` section 6): the working folder
   (`sites/<shortName>/runs/<yyyy-mm-dd>-<slug>.manifest.json`;
   `../references/runs/` in a legacy git checkout), then the source of truth's
   `runs/`, then ask whether a download exists from an earlier conversation.
   Never Agent Instructions. If one is `open`, offer `flows/resume.md` (resume
   or clean up) instead of a duplicate build.
5. **Designer probe**, once: `designer_tool > get_current_page`. Success means
   the Bridge App is running (snapshots, folder creation, canvas navigation);
   call `get_all_breakpoints` once, keep the list for Phase 6, and say so if it
   differs from the conventions. Failure is not an error: run the foreground
   procedure below, then record the answer for the run and degrade as Phases
   5 and 6 say.

   **Designer foreground procedure.** The Bridge App answers only while its
   Designer tab is open and in front. The failed response carries the launch
   link ("Launch the app using following link <url>"); take it from the
   response every time. It is a credential, different per operator and site,
   and never stored in the repository, a manifest, or a run record (rule 15).
   - Claude Code or Codex: `open "<url>"` (`xdg-open` on Linux), wait about 40
     seconds, re-probe; up to three attempts.
   - Cowork: `open` runs in the cloud container and cannot reach the Mac. Use
     a Mac-side browser tool if the session has one (for example Control
     Chrome's `open_url`; one re-probe after about 40 seconds worked on
     2026-09-22), otherwise do what Claude.ai does.
   - Claude.ai: show the url as a markdown link, ask the user to click it and
     keep the tab in front, re-probe once they confirm.

   Record reachable or not, and after how many attempts, in the run record; it
   decides whether Phase 6 snapshots run.

## Phase 1: Intake and design digest

Inputs, best first:

1. Claude Design **standalone HTML** or **zip export**. (The "Send to Claude
   Code" bundle works only on Claude Code and its format is unpublished; HTML
   or zip is canonical.)
2. Design canvas files from Claude Code (`.dc.html` artboards).
3. Other HTML/CSS; Figma exports as HTML or images.
4. Screenshots, PDFs.
5. Links, fetched only if the surface allows it and the link is public;
   otherwise ask for an export.
6. Copy documents and an image list with public URLs or Webflow asset names.

Write a **design digest** in chat, every line labelled `observed` or
`assumed`:

- ordered sections with a type guess: hero, logo bar, feature grid,
  testimonial, pricing, FAQ, CTA band, footer, or "unclear";
- per section: headings and levels, body copy, CTAs (label, destination),
  media (source, alt, intrinsic size), layout intent, states;
- the **token reconciliation table**: colors, fonts, sizes, spacing, radii,
  shadows against the site variables in the catalog and conventions, each
  `exact`, `near` (name the substitute), or `none` (flag it). Template styling
  wins; the design's values are evidence of intent, not instructions;
- interactions and motion: all go to the manual handoff list, because Webflow
  Interactions cannot be created through MCP (`../references/unsupported.md`);
- missing assets and placeholder copy (lorem, "TBD", "[image]").

Keep the digest shorter than the design. Screenshot-only input yields more
`assumed` lines; the interview closes them.

## Phase 2: Interview

Topics, order, and skip logic: `../references/interview.md`. Binding: at most
**3 questions per turn**; never ask what the digest or an earlier answer
settles; once a family is likely, offer **"use family defaults"** (SEO
patterns, schema type, folder, nav and footer, slug pattern from the entry);
finish with a six-to-ten-line summary and wait for a yes before Phase 3.
Answers go in the brief under `answers` and are copied into `page` and
`sections`.

## Phase 3: Template selection

Empty catalog: stop. Say the site has not been onboarded and there is no
family to build from; point the maintainer at `flows/onboard.md`. Never
invent a family.

1. Score each usable family against the digest (section overlap, CTA pattern,
   layout, content type); one line of reasoning per family.
2. Present the **top three** (or all, if fewer): purpose, section outline,
   template model, fit score, reference image from the entry, and what would
   be **Reused**, **Adapted**, or **New** under that family.
3. Recommend one. If it has `status: proposed`, say: "This family was
   proposed by onboarding and has not been confirmed by a maintainer; the
   build can proceed, and the report will say the family is unconfirmed."
   Read the status from the entry every run; drop the caveat once it is
   `promoted`. The user may:
   - pick another family;
   - override the template model (`duplicate-master`, `component-recipe`,
     `hybrid`), recorded in the brief;
   - choose isolation. Offer `branch` **only** when the conventions record
     branching as available, and then repeat whatever they still list as
     UNMEASURED as a warning (usually: the read path on a branch page is
     verified; writes on a branch page and `create_branch` without the
     Designer are not, until the first branch run; long-lived branches
     conflict). Never offer it when unavailable or unmeasured;
   - propose a **new family**: `flows/maintain.md` (Create a family), with
     maintainer confirmation before this build continues. On Claude.ai, hand
     the user a note for the maintainer and stop.
4. **Reconcile the chosen family**: resolve its component names to ids. This
   is the only component read before Phase 5. Resolve every name in the
   section outline and shell tables except `loose` rows (their class paths are
   checked after `create_page`) and `candidate:<slug>` rows (nothing exists
   yet):
   - components that share a group: **one**
     `data_component_tool > query_components`, a single labelled query whose
     `keywords` name the group and whose `limit` covers it;
   - otherwise one batched `data_component_tool > get_component` call by
     `name` (add `group` where a name needs it).

   Never `get_all_components` here, and never `includeProps`,
   `includeVariants`, or `includeInstanceCount`: those belong to the
   onboarding census and the maintain flow's impact analysis. Where the
   per-minute budget binds, prefer the one-call group query over many by-name
   lookups.

   Two guards, each **blocks that family** this run (offer the next best;
   point the maintainer at `flows/maintain.md`, Refresh inventory, then fix
   the entry):
   - **zero** matches: "The catalog names `<name>`, which no component on the
     site matches; `<family>` is blocked for this run."
   - **more than one**: "The catalog names `<name>`, which matches <n>
     components on this site (groups: <groups>); `<family>` is blocked until a
     maintainer disambiguates the row by group or renames one of the
     components." On a site where onboarding found every name unique
     ("Component names"), this is a drift alarm.

   Keep the name-to-id map for the run; Phase 5 writes it to the manifest. The
   catalog never stores ids, so a recreated component only has to keep its
   name. A resumed run re-resolves.

**Reuse labels**, exactly one per section: **Reused** (existing component or
variant, values only), **Adapted** (new variant or prop on an existing
component), **New** (component created this run; becomes a candidate). A
master section copied as a loose tree (`Component name` = `loose`) is always
Reused; to change its layout, replace it with a component or a New section.

## Phase 4: Visual outline (approval gate)

1. **Brief.** Assemble it per `../references/brief-schema.md` (worked example
   `../assets/examples/example.brief.json`, minimal
   `../assets/brief.example.json`). Outline and build read this one object, so
   what is reviewed is what gets built. Mark every slot the build cannot write
   as `manual`, per the conventions' capability table ("Rate limits and
   response sizes"): on a site whose element reads exceed the budget, that is
   loose-section text, links, and images and every slot child; on a small site
   maybe nothing.
2. **Validate:** `python3 scripts/validate_brief.py brief.json`; fix every
   violation before rendering (no scripts: the checklist in
   `brief-schema.md`).
3. **Render:** `python3 scripts/render_outline.py brief.json > outline.html`
   (no scripts: `../references/outline-spec.md`). One self-contained HTML
   file: desktop plus three narrower frames mapped onto the site's recorded
   breakpoints (say which); sections in order with reuse label, component and
   variant, slots with status (`manual` reads "publisher enters in the
   Designer"); open decisions; token substitutions; the manual handoff list,
   opening with the count of `manual` slots.
4. **Show it** (an artifact on Claude.ai; open the file on Claude Code and
   Codex) with a **section mapping table** in chat: design section → family
   section → component/variant → content → status.
5. Iterate until the user says **approved**; record `approvedAt`. Approval
   covers the outline and every proposed addition (variants, components,
   variables). After it, routine implementation choices need no confirmation;
   anything that changes the approved outline comes back as a question.

## Phase 5: Build

**Every Webflow write is appended to the run manifest before the next step**
(`python3 scripts/manifest.py append ...`, or by hand per
`manifest-schema.md`). Pace every element, props, settings, and builder call
as the conventions say (rule 10); batch actions per call.

1. **Open the run manifest.** `manifest.py create` with site, family and
   version, model, isolation, surface (`claude-ai`, `claude-code`, `cowork`,
   or `codex`), and the brief hash. **Write-ahead first**
   (`../references/stores.md` section 7): on Cowork, Claude Code, and Codex
   write the manifest and brief to the working folder
   (`sites/<shortName>/runs/<yyyy-mm-dd>-<slug>.manifest.json` and
   `.brief.json`; `../references/runs/` in a legacy git checkout) before
   anything else; on claude.ai hand both over as downloads at the end of every
   phase. Then write them to the source of truth and append every later step
   there too. **Never `create_instruction`**: a run record is not guidance. If
   a source-of-truth write fails, do not abort: mark the step `failed`, retry
   once at the end of the next phase, and say where the record lives
   meanwhile. The local copy survives a 429, a 403, or a closed tab.
2. **Pre-snapshot.** Names and definitions of every style, component, and
   variable the run may touch (the family's components with variants and
   props, the classes the catalog rows name, the variables in the
   reconciliation table), plus the full component list (`get_all_components`
   without props and without `includeInstanceCount`; the guard compares the whole list, so this
   is the one build read of every component). Shape:
   `manifest-schema.md`, "Snapshot shape"; trimmed example
   `scripts/tests/fixtures/example/pre.snapshot.json`. Serialize style reads;
   no instance counts. Save the JSON with the manifest (`preSnapshotHash`);
   Phase 6 compares against it.

   On Claude Code, large results (`get_all_components` with props and
   variants, `get_variables`, `query_styles` with `include_properties`) are
   saved to files. Build the snapshot from them with
   `python3 scripts/build_snapshot.py` (it maps tool output onto the shape,
   keeps only the named classes, expands the family's props and variants, and
   prints the sha256 for `manifest.py set --pre-snapshot-hash`). **Never
   retype an inline result**: a slip becomes a false guard verdict. If one
   comes back inline, repeat the call larger so it is saved (two variable
   collections per call; padding `name_path` queries to `query_styles`, which
   the script drops) and use the identical call for the post-snapshot. Do the
   brief's asset-library lookups **before** this step, and leave the
   prescribed gap between the last bulk read (`list_assets`,
   `get_all_components`, `get_variables`) and step 5's first element read:
   they share one per-minute budget (rule 10).
3. **Isolation.** `draft-main`: nothing. `branch`: `data_pages_tool >
   create_branch` from the master (duplicate-master, hybrid) or an empty page
   (recipe), poll `get_branch_task_status` until done, and use the branch page
   id for every later call. What is verified on a branch page is in the
   conventions. If a write rejects the branch page id, stop and report; never
   fall back to main silently.
4. **Page.** `data_pages_tool > create_page` with `title`, `slug`,
   `parentFolderId` (pre-created at onboarding), `seo`, `openGraph`,
   **`draft: true` set explicitly**, and `duplicateOf` = the master id for
   duplicate-master and hybrid. Immediately `get_page_metadata`; if
   `draft: true` does not read back, stop, `update_page`, and re-read before
   anything else.
5. **Sections.** One `data_element_tool > get_all_elements` at small depth (2:
   body, `main`, section roots) to map the duplicate onto the catalog rows.
   Pacing per rule 10 and the site's numbers: no bulk read in the preceding
   quiet window, the prescribed gap before the next element read, and on a 429
   one ten-minute zero-traffic wait and one retry; then a `failed` step naming
   the endpoint, the page left a draft, the manifest `open`, and a BLOCKED
   report (`flows/resume.md` continues from there). No full-depth read on an
   image-heavy page unless onboarding measured that it works: each image node
   resolves an asset. Read content nodes later, one section at a time, with
   `query_elements` scoped to the section root (step 6), where that tool works
   at all.

   Map the children of `main` in order: a component instance (match the id
   Phase 3 resolved for the row) or a loose root (match position and class
   path: the brief's `masterSection` and `classPath`). Then apply the outline:
   `remove_element` on dropped master sections and copied instances (allowed
   because this run's `create_page` made them);
   `data_component_builder > insert_component_instance` with the resolved id
   at the outline's position, slots filled with `insert_in_slot`;
   `move_element` to reorder. Never create a loose section new: a section the
   master lacks is a component or a New section. Read back
   `get_all_elements` once after the pass.
6. **Content.** Props via `data_component_props_tool >
   set_component_instance_prop_values`, batched (several instances per call
   are fine); text props take `type: string`. Link props may accept only
   `url`, `email`, and `phone` although the schema lists `page`: then keep the
   component's default page link or set it with `data_element_settings_tool`
   on the button inside the instance, and say which in the report.

   **Loose-section nodes and slot children** are automated only where the
   conventions' capability table says so. There: find nodes with
   `data_element_tool > query_elements` and an `element_filter` scoped to the
   section root (element type, class, or the master's text), and write with
   `data_element_settings_tool` (text, link `href` and target, image asset and
   alt), H1 first. Where it says not (deep reads 429, slot children without
   element ids), every loose slot and slot-child prop is a manual handoff
   item; component props are the only automated content path.

   **Links.** Destination and target come from the brief (a page id internal,
   a URL path external) and are copied as given. **Extras** (UTM keys, custom
   attributes) are copied **only** where they are the site's measured
   convention (conventions, "Site-level tracking"): the same keys or attribute
   other CTAs on the site already carry. Anything else is a new tracking scheme
   (rule 14): do not invent extras the convention does not list, and do not
   silently keep brief-only extras that deviate from it. Where the brief's
   extras **differ** from the convention, stop and ask in that same turn:
   "This destination carries `<brief extras>` but the site's other CTAs use
   `<convention extras>`. Shall I apply the site's extras, keep the brief's
   extras (a deviation from the convention, which the report will name), or
   hand this link to the publisher?" Write nothing deviant before the answer.
   Convention `none` or UNMEASURED: destination and target only. Never add
   extras to be helpful, and never create or insert an analytics component
   (one the family's shell table does not name is Designer work; one it names
   but the page lacks is a Phase 6 HANDOFF).

   **Repeated nodes.** The master has more than the brief (a fourth stat
   card): `remove_element` the surplus. Fewer: insert with
   `data_element_builder` (loose) or `insert_in_slot` (slot), and say so.
   **Images:** prefer an asset already in the library, matched by name
   (`data_assets_tool`); alt text on every image; internal links as page ids,
   never typed paths. Slots marked `missing` get a visible
   `TODO: <slot name>` placeholder.

   **Before any `asset_tool > upload_image_by_url`, stop and warn, in these
   words:** "Uploading an asset makes it public immediately.
   `upload_image_by_url` puts the file in the site's asset library, and
   Webflow serves library assets from a public CDN URL
   from the moment of upload, before any publish and whether or not this
   draft page is ever published. Anyone who has the URL can fetch it without
   logging in. Deleting the asset later does not un-serve a URL someone
   already has. May I upload `<file or source url>`, or is there an asset
   already in the library I should use instead?" Wait for the answer. Never
   upload anything confidential, unreleased, or under embargo to get a draft
   built (rule 19). Record every upload in the manifest; Phase 7 lists them as
   **already public**.
7. **Adapted sections.** A new variant (`data_component_variants_tool >
   create_variant`, then `set_variant_styles`) or new props
   (`data_component_props_tool`). Never edit base variant styles of a
   pre-existing component.
8. **New sections.** `data_component_tool > create_blank_component` with
   `group` = family slug and `description` = `<family>@<version> | candidate |
   run <slug>`. Build inside it with `data_element_builder`, or with
   `data_whtml_builder` only after remapping the CSS to existing classes and
   variables (single root, no `<style>`, no `@keyframes`, only the breakpoint
   media queries the conventions record). Create styles (`data_style_tool`)
   before the elements that use them; new class names follow the site's
   naming system with the family prefix. Write a candidate record
   (`../references/catalog/candidates/README.md` format) as
   `sites/<shortName>/candidates/<slug>.md` to the working folder and the
   source of truth, never to Agent Instructions.
9. **Page-level metadata.** JSON-LD via `data_pages_tool >
   bulk_update_pages_schema_markup` from the family's schema template, **if**
   the conventions record the schema write as allowed for this role. If it is
   refused (**403 `insufficient_permissions`**, access and entitlement table):
   mark the step `failed`, hand the JSON-LD to the publisher (Page settings >
   Custom code), and do not put `FAQPage` in an FAQ component's `Schema` prop
   unless the visible question blocks match it. No page scripts. Custom embeds
   only if the approved outline lists them; otherwise a handoff note. Every
   Adapted or New section has its candidate record (step 8); if the source of
   truth is unreachable, it stays next to the manifest and goes to the
   maintainer with the report.
10. **Never**, in any phase: `publish_site`; `publish_branch` (except staging,
    on explicit request, in branch mode); `unregister_component`;
    `delete_variable`; `remove_element` on anything this run did not create;
    `update_style` on a pre-existing class; `set_site_scripts`;
    `set_page_scripts`; localization writes; link extras outside the measured
    tracking convention; creating or inserting an analytics or tracking
    component.

If the Designer is unreachable and a step needs it (folders, canvas
navigation), do not improvise: use the pre-created folder, or record a manual
handoff item.

## Phase 6: Verify (normative checklist, readback not memory)

Every check reads the live site, never intent. Reads first; the checklist is
filled from what they returned.

- **Structure.** One `data_element_tool > get_all_elements` on the page
  (serialized, never alongside other element reads), as deep as the
  conventions allow: section order matches the outline; instances and
  variants match the recipe; props are set; no leftover master sections;
  loose roots keep the catalog's class paths. Everything else uses the cheap
  reads (`get_page_metadata`, `get_all_components`,
  `query_pages_schema_markup`).
- **Page.** `get_page_metadata`: `draft: true`, slug, folder, SEO, Open Graph;
  the schema type reads back.
- **Guard.** Post-snapshot exactly as Phase 5 step 2, then
  `python3 scripts/diff_inventory.py pre.json post.json`. Any change to a
  **pre-existing** definition fails the run: name each in the report and ask
  whether to revert it by hand in the Designer or accept it (a deliberate
  deviation, `diff_inventory.py --accept kind:key`). Save `postSnapshotHash`
  and `guardVerdict` to the manifest.
- **Content.** No lorem or TODO except slots marked `missing` (visible
  `TODO:`, listed); one H1 and no skipped levels; alt on every image; links
  resolve (page ids exist, URLs well-formed); the primary CTA in the first or
  second section.
- **Visual.** Designer reachable: `designer_tool > switch_page` to the page,
  then `element_snapshot_tool` on the root and each section, **one call at a
  time** (parallel calls return `status: false`). After the foreground
  procedure the Bridge App sits on the home page in a new tab: `switch_page`
  first, or snapshots return `status: undefined`. Iframes render blank. A
  snapshot shows the canvas's current breakpoint (the tool takes no viewport);
  for narrower views ask the human to switch the breakpoint (ids from the
  conventions) and snapshot again. Element ids come from the structure read;
  if that was blocked, only `designer_tool > get_selected_element` on an
  element the human selects works. Otherwise report: "Visual verification
  pending. Open the page in the Designer with the Bridge App running and
  re-run verify."
- **Ships-at-next-publish and already-public lists.**
  `python3 scripts/manifest.py ships manifest.json` returns both.
  `shipsAtNextPublish`: components, styles, and variables this run created on
  main; empty for `branch`. `alreadyPublic`: every asset this run uploaded, in
  **both** isolation modes, because the asset library is site-level and
  serves a public CDN URL from upload (rule 19). Without the script, read
  `created.styleNames`, `created.componentIds`, `created.variableIds`, and
  `created.assetIds` from the manifest (`manifest-schema.md`).

### The normative checklist

Fill it once, from the reads above, before the report. Six rows, always, in
this order. Verdicts:

- **PASS**: read back and it matches.
- **WARN**: could not be checked (a read was blocked, the Designer was closed,
  the convention is UNMEASURED); say what was skipped and why.
- **FAIL**: read back and it does not match; the run is not `verified`.
- **HANDOFF**: a gap the publisher closes in the Designer; it does not fail
  the run and is repeated in the manual work list (`unsupported.md` wording).

Evidence is what was read: the call, the node or field, the value. "Built as
planned" is not evidence, nor is the brief; a row whose evidence would come
only from memory is WARN, not PASS.

| id | Check | PASS when | Evidence |
| --- | --- | --- | --- |
| `outline-match` | Outline match | Section order equals the approved outline; each section's component and variant equals the recipe; no master section the outline dropped is still on the page | The `get_all_elements` read: children of `main` in order, with component id and variant per instance and the class path per loose root |
| `family-rules` | Family rules | Every `required` row of the family's section outline is present; the shell components the brief kept (nav, footer, and an analytics component if the family's shell table names one) are on the page; the family's checkable Do items hold — one primary CTA, an H1 present | The same structure read, matched against the catalog entry's Section outline `Required` column and Shell components table |
| `cta` | CTA | The primary CTA is above the fold (first or second section); every CTA label and destination equals `answers.primaryAction` and the brief's `ctaLabel` / `ctaLink` slots | Prop values and `data_element_settings_tool` link values as read back, quoted next to the brief's values |
| `seo` | SEO and page metadata | `draft: true`, slug, parent folder, SEO title and description, Open Graph, and schema type all read back as the brief specifies | `get_page_metadata` and `query_pages_schema_markup` |
| `guard` | Guard | `diff_inventory.py` reports no change to a pre-existing style, component, or variable definition | The guard verdict and, on a change, the named key |
| `tracking` | Tracking | Every CTA carries the link extras the site's measured convention expects, and the family's analytics shell component is on the page | The live link values from the same structure read, next to the convention recorded in `webflow-conventions.md`, "Site-level tracking" |

Row rules where the verdict is not the obvious one:

- `family-rules`: a missing analytics shell component is **HANDOFF**, never a
  silent create; the publisher places it.
- `cta`: a destination that does not match the brief, or a missing label, is
  **FAIL**: wrong content, not a gap.
- `guard`: **FAIL fails the run**; nothing in this table softens it.
- `tracking`: **HANDOFF only**; never FAIL a run for a missing UTM parameter or
  attribute. Compare the brief's `primaryAction` and `ctaLabel` / `ctaLink`
  slots with the live links, against the convention recorded at onboarding
  (delivery: site scripts, a named shell component, per-page embeds, none;
  visible extras: none, UTM keys, an attribute name). Either answer
  UNMEASURED: **WARN**, "tracking convention unmeasured; skipped". Never infer
  GTM events or click listeners; the MCP cannot read them. The build adds no
  tracking (rule 14); gaps go to the publisher in the conditional wording of
  `unsupported.md`.

Record the table with `python3 scripts/manifest.py set --normative-checklist
checklist.json` (`manifest-schema.md`, `normativeChecklist`), or by hand. A
failed guard sets the manifest `status` to `failed` until the user decides; a
clean run sets `verified`. A `tracking` HANDOFF does not change the status.

## Phase 7: Report and handoff

In this order:

1. **Where the page is**: page id, slug, and folder path in plain text
   (`Pages panel > <folder> > <page>`). With the Bridge App connected, call
   `designer_tool > switch_page` to it and say so. Offer a
   `https://<site-short-name>.design.webflow.com?pageId=<page id>` link only
   if the conventions record that it works on this site; otherwise mark it
   "unverified".
2. Isolation mode (and branch name, if any).
3. The section mapping table **as built**, with reuse labels.
4. **The normative checklist table first**: all six rows, verdict and
   evidence, in the Phase 6 order; then the other verification results and
   the guard verdict.
5. Manual work list: Interactions, embeds, fonts, anything from
   `unsupported.md`, every `manual` slot, steps skipped because the Designer
   was closed, and every HANDOFF row, worded as Designer work.
6. Open decisions still unresolved.
7. **Already public: assets uploaded by this run.** One line per asset, name
   and CDN URL: "these files are on Webflow's public CDN now, before any
   publish and whether or not this page is ever published; anyone who has the
   URL can fetch them without logging in. Deleting an asset later does not
   un-serve a URL someone already has." If nothing was uploaded, say "none:
   every image came from the existing asset library". This line comes before
   the ships-at-next-publish list: that one is a prediction, this one has
   already happened.
8. Ships-at-next-publish list, in plain words: "these will go live for the
   whole site when someone next publishes, even though the page stays a
   draft."
9. Candidate records written (`candidates/<slug>.md`, with store and page or
   path), and that a maintainer promotes them through `flows/maintain.md`. If
   the family is `status: proposed`, say its family, master, and component
   names have not been confirmed by a maintainer.
10. **Where every record went**: brief, manifest, outline, snapshots, guard
    verdict, each with its store (working-folder path, Notion page, or
    download), and where guidance was read from (source of truth, mirror, or
    bundle; the mirror unreadable for this role, if so). On claude.ai also
    hand over the brief and manifest as downloads. If the source of truth was
    unreachable: "The run record and the brief exist only as these files. Keep
    them; resume and cleanup need them, and a maintainer can file them into
    `<store>`."
11. Publishing instructions, per the conventions' "Publish policy": the
    publisher finishes the manual list, reviews in the Designer, turns off the
    draft flag, and publishes (or merges the branch) themselves. The skill has
    not published and will not.

Close the manifest (`verified` or `failed`) in the working folder and update
the source-of-truth copy before ending the conversation.
