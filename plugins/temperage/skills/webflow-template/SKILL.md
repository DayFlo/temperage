---
name: webflow-template
description: >-
  Turns a design (Claude Design HTML or zip export, other HTML, screenshots, a
  PDF, or a link) into an unpublished draft page in your Webflow site, built
  from a reusable template family: it digests the design, interviews the
  submitter, picks a family, shows a visual outline for approval, builds the
  page through the Webflow MCP connector, verifies by readback, and reports.
  Also carries the maintainer flows: onboard a site (run this first; the
  catalog is generated per site), check access, maintain the template catalog,
  sync the catalog to its Webflow Agent Instructions mirror, resume or clean up
  an interrupted run. Never publishes. Runs only when the user asks for it by
  name (/temperage:webflow-template in Claude Code and Cowork,
  "webflow-template" selected in Claude.ai, $webflow-template in Codex). Do
  not activate it for general design or Webflow conversation.
license: MIT. See LICENSE
metadata:
  version: "1.1.1"
disable-model-invocation: true
---

# Webflow template build

You are running the `webflow-template` skill. It turns a design into a draft
page in a Webflow site by reusing a **template family**: the components,
styles, variables, and layout conventions already in that site. The template's
styling wins over the incoming design. The page it builds is not visible to
visitors until a human publishes. One thing it can do, uploading an asset, is
public the moment it happens: see "What this can and cannot make public"
below.

The shape of every build is fixed:

**receive a design → digest it → interview the submitter → pick a family →
approve a visual outline → build an unpublished page → verify by the
normative checklist (readback, not memory) → report and hand off.**

## Onboarding comes first

The skill ships **without a catalog**. A template family names components,
master pages, and folders that exist on exactly one site, so there is no
portable catalog to ship. `flows/onboard.md` reads a site once and generates
the catalog, the filled conventions, the index, and the sync state for it
(`references/stores.md` section 2 is the layout: `sites/<shortName>/catalog/`,
`conventions.md`, `catalog/index.md`, `sync-state.json`, plus a local site
inventory that is never shared; step 0 of that flow picks the store). Until
that has run, a build gets as far as template selection and stops.

Everything site-specific lives in `references/webflow-conventions.md`, which
ships as a **template full of placeholders**. Rows marked MEASURE are
onboarding's worklist; rows marked UNMEASURED are unknown and no flow may
assume them. A filled example for a fictional site is in
`references/examples/`.

**Where things live** (`references/stores.md`). Two kinds of data, two homes:

- **Guidance** (the rulebook, the catalog entries, the index, the conventions)
  lives in the **source of truth** and is **mirrored** into Webflow Agent
  Instructions under `<prefix>` whenever the connector user's site role
  allows, because that mirror is the one store every MCP client on the site
  reads.
- **Records** (briefs, run manifests, snapshots, guard verdicts, candidates,
  sync state) live in the source of truth only. They are **never** written to
  Agent Instructions: they are not guidance, they would inflate every agent's
  instruction discovery on the site, and that store has no history.

The source of truth is one of **Notion** (through the Claude Notion
connector), a **working folder**
(Cowork, Claude Code, Codex: a plain folder, a synced folder, or a git
checkout, which is the only path with a pull request), or **downloads only**
(claude.ai with no document connector). An administrator configures it once for
the organization (`references/org.json`, or the plugin's configuration prompt),
and onboarding writes a **pointer** to it into `rules/<prefix>.md` on the site,
so every later agent on the site finds it without a setup question. The
working folder is also the always-on **write-ahead** layer: a record is written
there, or handed over as a download, before any network store. Onboarding does
not need a repository; the measuring is identical on every store, and every
store also hands the user a download bundle.

**What a page store gives up, and say it rather than gloss it:** there is no
pull request unless the working folder is a git checkout. What stands in for
one is the page's own version history and comments, an explicit confirmation
before each guidance write, families kept as `status: proposed` until a
maintainer confirms them, a `## Decisions` section inside every entry, and the
bundle as a portable copy. The detail is in `flows/onboard.md` step 0 and
"Maintaining in a page store" in `flows/maintain.md`.

## Surfaces

| Surface | How it is invoked | Webflow access | Scripts | Persistence (`references/stores.md`) |
| --- | --- | --- | --- | --- |
| Claude.ai | User selects "webflow-template" by name | First-party Webflow connector (OAuth per user) | Code execution sandbox, Python only if enabled | **No repo, and none needed.** Source of truth through the Notion connector when it is connected, else downloads only (records handed over at the end of every phase); guidance mirrored into Webflow Agent Instructions under `<prefix>` when the role allows; a download bundle always |
| Claude Code | `/temperage:webflow-template` (plugin skills are namespaced; the bare form only applies to a copy in `.claude/skills`) | Plugin `.mcp.json` (`https://mcp.webflow.com/mcp`) or the project's MCP | Local `python3` | Working folder as write-ahead, then the organization store; a git checkout as the working folder gives a pull request, the reviewed path |
| Cowork | The Skill tool with the namespaced name `temperage:webflow-template` (the skill is not listed among available skills, because of `disable-model-invocation: true`, but explicit invocation works), or `/temperage:webflow-template` | The claude.ai Webflow connector. The plugin's `.mcp.json` server does not load in Cowork; the claude.ai connector serves every call | `python3` 3.11 in the Cowork cloud container, else the written fallback | The Cowork session runs in a **cloud container** (`~` is `/root`), not on the user's Mac: a working folder there is write-ahead for the session only and is not visible on the Mac unless the user connects a folder. So on Cowork the source of truth is Notion (the connector's file upload and attachment tools are available) or downloads, and a connected folder is the only persistent working folder. The plugin directory is writable in Cowork (a synced copy); never write there anyway. Hooks and sub-agents are Cowork-only features; this skill ships neither |
| Codex | `$webflow-template` | `[mcp_servers.webflow]` in `~/.codex/config.toml` | Local `python3` | Working folder; the organization store only when an MCP server for it is configured |

Submitters on Claude.ai are often non-technical. Ask short questions, at most
three per turn, and explain Webflow terms the first time you use them.

## Administrator configuration

Claude Code and Cowork substitute `${user_config.*}` tokens in this file only,
never in the flows or references they read from disk. This table is therefore
the one place the plugin configuration reaches the skill. A cell that still
reads as a literal `${user_config.…}` token, or is empty, is unset: fall
through to `references/org.json` and the rest of the discovery order
(`references/stores.md` section 6).

| Setting | Value in this conversation |
| --- | --- |
| source of truth | ${user_config.source_of_truth} |
| Notion parent page id | ${user_config.notion_parent_page_id} |
| working folder | ${user_config.working_folder} |
| instruction prefix | ${user_config.instruction_prefix} |
| allowed stores | ${user_config.allowed_stores} |
| sites | ${user_config.sites} |
| tested MCP version | ${user_config.tested_mcp_version} |

## Preflight (every conversation)

1. Call `webflow_guide_tool` once per conversation. Do not call it again.
   Record the version string it returns and compare it with the tested version
   in `references/org.json` (`testedMcpVersion`) or the conventions file
   ("Tested MCP version"); when they differ, say so once. Webflow changes error
   codes and role rules, and the skill should notice drift before a user does.
   The response can exceed the client's tool-result limit (it does in
   Cowork); the client then saves it to a file, so read the version and the
   session id from that file rather than calling the tool again. If the
   response issues a **session id**, keep it and pass it to every later
   Webflow call that accepts one; a call made without it may be refused.
2. Find the source of truth and read the guidance. Run the **discovery order**
   in `references/stores.md` section 6: `data_agent_instructions_tool >
   search_instructions` once with no filter (a `rules/<prefix>.md` hit whose
   first fenced block is a `webflow-template` pointer names the store), then
   the administrator configuration (the table above, then
   `references/org.json`), then the working folder's cached `org.json`, then
   ask once. `<prefix>` is the instruction prefix recorded once in
   `references/webflow-conventions.md`, "Toolkit settings"; the default is
   `page-templates`. Then read the rulebook, the index, the conventions, and
   the catalog entries **from the source of truth** through its adapter; else
   from the Webflow mirror (`read_instruction` on `<prefix>/SKILL.md`,
   `<prefix>/conventions.md`, each `<prefix>/catalog/<family>.md`; say the
   mirror may lag); else from an attached bundle (say its `generatedAt`); else
   fall back to the bundled `references/rules.md` and the empty
   `references/catalog/` and tell the user the site has not been onboarded.
   If the Webflow call returns anything but 200, classify it by the **access
   and entitlement table** in `references/unsupported.md` and repeat that
   row's exact ask: a 403 `forbidden` on instructions is a Webflow
   **site-role** gate (Reviewer and custom roles cannot read Agent
   Instructions), not a missing OAuth scope. The mirror is then neither read
   nor written for the conversation and the report says so. A 403 never aborts
   a run and is never retried in a loop: one probe, one answer. When no store
   is readable **and** no bundle is attached, ask for the bundle onboarding
   produced, or send the user to `flows/onboard.md`; `flows/build.md` Phase 0
   step 2 has the wording and says what is degraded. Records (the brief, the
   manifest, candidates) go to the working folder first, then the source of
   truth, never to Agent Instructions.
3. Follow `references/rules.md`. It is short. Every rule in it is binding on
   this skill and on any other agent connected to the site.
4. Read `references/webflow-conventions.md` before planning any reads. It is
   where this site's measured facts live: the instruction prefix; the pacing
   every element, props, settings, and builder call needs, which follows from
   the site's CMS collection and asset counts (rule 10); whether loose-section
   content and slot children are reachable at all or are manual handoff items;
   whether the connector user's role can write page schema; whether branching is
   available; the breakpoint list; whether component names are unique. None of
   these is a constant: a small site may hit no limits at all, a large one may
   hit several. Where the file says UNMEASURED, say so instead of guessing, and
   point the maintainer at `flows/onboard.md`. The catalog names components
   rather than carrying their ids, so a run resolves only the names of the
   family it picked, with one `query_components` filtered to that family's
   component group or batched `get_component` by name, and records the ids in
   the run manifest alone (rule 10, `references/catalog/README.md`). A catalog
   family with `status: proposed` is unconfirmed until a maintainer confirms it
   through `flows/maintain.md`, and the report says so while that is true.
5. The Designer tools (`designer_tool`, `element_snapshot_tool`) only answer
   while the Webflow Designer is open with the Bridge App running and its tab
   in the foreground. When `get_current_page` fails, its error response carries
   the Bridge App launch link for the connected account and site ("Launch the
   app using following link ..."). Take the link from that response, never from
   a file: it is a credential and it differs per operator. On Claude Code or
   Codex run `open` (macOS) or `xdg-open` (Linux) on it, wait about 40 seconds,
   re-probe, up to three times. On Cowork the session runs in a cloud
   container whose `open` cannot reach the user's Mac: use a browser tool the
   session has on the Mac side (for example Control Chrome's `open_url`), wait
   about 40 seconds, and re-probe the same way; without one, do what Claude.ai
   does. On Claude.ai show it as a markdown link and ask the user to click it. Only then record the Designer as unreachable for the
   run. Never write the link to a file, a run record, or a commit.

## Pick a flow

The **build** flow is the default. If the user hands you a design, a link, or
says "make a page", go to `flows/build.md` without asking which flow they mean.
The maintain and sync flows are for maintainers and are usually run from
Claude Code; onboarding runs anywhere:

| User wants | Flow | Who |
| --- | --- | --- |
| Set the site up for the first time: inventory, families, conventions, install; or configure the store for an organization | `flows/onboard.md` | Anyone with the Webflow connector, once per site, **before anything else**; an administrator once per organization |
| "Check access": which Webflow capabilities and which store this account can reach, with the exact ask for each refusal | `flows/doctor.md` | Anyone; onboarding and build run it themselves |
| A page from a design (default) | `flows/build.md` | Anyone |
| Confirm a proposed family, promote a candidate, edit a master or shared component, create a family, refresh inventory, prune and archive | `flows/maintain.md` | Maintainer |
| Push guidance from the source of truth to the Webflow mirror; pull guidance edits made in the Webflow Instructions panel back | `flows/sync.md` | Maintainer |
| Finish or clean up an interrupted run | `flows/resume.md` | Anyone; build preflight offers it automatically |

Read only the flow you are running. Each flow names the reference files it
needs. Keep file references one level deep: this file points to a flow, the
flow points to references.

## What this can and cannot make public

Using this skill must not put anything on the public internet. Only one of the
writes it makes can, and it says so at the moment it makes it. The full rule is
`references/rules.md` rule 19.

| What | Public? | The guarantee |
| --- | --- | --- |
| **The draft page** | No | Created with `draft: true` set explicitly and confirmed by `get_page_metadata` readback. Draft pages are excluded from publishing, so it does not go live at the next site publish either. A human turns the flag off. |
| **New components, styles, variables** | Not yet - **at the next site publish, yes** | They are site-level, so they ship whenever anyone next publishes the site, even though the page stays a draft. Every run reports them as the ships-at-next-publish list. Branch mode keeps them off main until merge. |
| **Uploaded assets** | **Yes, immediately** | `asset_tool > upload_image_by_url` puts the file in the site's asset library, and Webflow serves library assets from a public CDN URL from the moment of upload, before any publish and whether or not the page is ever published. The build warns and asks first, prefers an asset already in the library, and reports every upload as an "already public" line. Deleting an asset later does not un-serve a URL someone already has. |
| **Branch staging publish** | Gated, not open | Only on explicit request in that turn, only in branch mode, only to staging, never production. An anonymous request to a Webflow branch staging URL redirects to the Webflow login and returns HTTP 403. |
| **Agent Instructions** (the guidance mirror; never a record) | Evidence says no; no vendor statement | Gated by Webflow site role (Site manager and Designer manage; Marketer and Content editor read; Reviewer and custom roles cannot read), delivered to authorized MCP clients as site metadata, no publish path, never in page content. Not a guarantee: `flows/onboard.md` step 10 runs a one-time check per site (throwaway instruction, human publishes on their own cadence, confirm the marker appears nowhere public, delete it). |
| **CMS items** | Never used | Standing non-goal. CMS items have staged and live states with publish and unpublish events; they are publish-shaped by design, so the toolkit never stores a catalog, brief, candidate, or run record in a collection. |

Never `publish_site`, in any flow, on any surface.

## Hard safety rules

These apply in every flow, on every surface, with no exceptions and no
"just this once". The full rulebook is `references/rules.md`; these are the
lines that can do damage if crossed.

- **Never call `publish_site`.** `publish_branch` only to staging, only in a
  run that uses branch mode, only when the user asks for it in that turn, and
  always recorded in the manifest. Production publishing is done by humans.
- **Every page is created with `draft: true` set explicitly** (the API default
  is `false`) and confirmed by `get_page_metadata` readback before any other
  write to the page.
- **Never edit a pre-existing shared definition.** No `update_style` on a class
  that existed before the run, no base-variant edits on a pre-existing
  component, no changes to existing variables. Extend with a new class, a new
  variant, or a new variable instead. The pre/post inventory guard fails the
  run if anything pre-existing changed.
- **Never delete or unregister anything not created in the current run.**
  Destructive calls on run-created resources require the user's explicit
  confirmation in that turn.
- **Record every Webflow write in the run manifest before the next write.** On
  interruption, resume from the manifest instead of rebuilding.
- **Touch only toolkit-owned Agent Instruction paths, and only with
  guidance**: `rules/<prefix>.md`, `<prefix>/SKILL.md`,
  `<prefix>/conventions.md`, `<prefix>/catalog/<family>.md`. Never overwrite
  instructions the toolkit does not own, and never write a brief, manifest,
  snapshot, candidate, or sync state there: records go to the working folder
  and the source of truth (`references/stores.md`).
- **Write records locally first.** The working folder (or the phase-end
  download on claude.ai) gets the manifest and brief before any network store
  does, and never a path inside the plugin directory.
- **Warn before uploading an asset, and prefer one already in the library.**
  An uploaded asset is served from a public CDN URL from the moment of upload,
  before any publish. It is the only thing this skill does that puts a file on
  the public internet.
- **Never write a credential to a file.** The Bridge App launch link is read
  from a failed probe response at run time and is never stored.
- **No site or page scripts, no localization writes, no publishing of any kind
  from this skill.**

## Scripts and the no-script fallback

Six Python 3 scripts live in `scripts/`. They use the standard library only,
take JSON in, return JSON or HTML out, and exit non-zero on failure.

| Script | Purpose | Flow |
| --- | --- | --- |
| `scripts/validate_brief.py` | Check the brief before rendering the outline | build Phase 4 |
| `scripts/render_outline.py` | Render the self-contained HTML outline from the brief | build Phase 4 |
| `scripts/diff_inventory.py` | Pre/post inventory diff; the shared-defaults guard | build Phase 6 |
| `scripts/catalog_lint.py` | Lint catalog entries and `sync-state.json` | onboard, maintain, CI |
| `scripts/manifest.py` | Create, append to, summarize the run manifest; ships-at-next-publish list | build, resume |
| `scripts/build_snapshot.py` | Build the guard's pre/post snapshot from saved `get_all_components`, `get_variables`, and `query_styles` results | build Phases 5 and 6 |

Run them with `python3 scripts/<name>.py --help` first if unsure of arguments.

**When code execution is unavailable** (Claude.ai with the sandbox off, or a
script fails to run): do the same work by hand from the written spec. The specs
are exact: `references/brief-schema.md` carries the validation checklist,
`references/outline-spec.md` the rendering rules,
`references/manifest-schema.md` the append and summarize procedures, and
`references/rules.md` rule 17 the guard. Say in the report that helper scripts
were unavailable and the checks were performed manually. Never skip a check
because the script could not run.

## Files

Flows (procedures, read one at a time):

- `flows/onboard.md`: once per site; inventory, measurements, families,
  conventions, install; the "configure for organization" path. Run first.
- `flows/doctor.md`: "check access"; one read-only probe per capability and
  per store, classified, with the exact ask for each refusal.
- `flows/build.md`: the per-page workflow, Phases 0 to 7.
- `flows/maintain.md`: confirm proposed families, promote candidates, edit
  shared components, create families, refresh inventory, housekeeping.
- `flows/sync.md`: source of truth to Webflow mirror push, mirror to source of
  truth pull for guidance edits, the pointer block, conflict rules.
- `flows/resume.md`: resume or clean up an interrupted run.

References (read when a flow points at them):

- `references/stores.md`: **the store specification.** Data kinds, the one
  layout, the adapters (Notion, working folder, downloads, Webflow mirror), `org.json`, the Webflow pointer block, the discovery order, failure
  and concurrency rules.
- `references/org.json`: the organization configuration, shipped empty; an
  administrator fills it (or answers the plugin's configuration prompt).
- `references/rules.md`: the Webflow MCP rulebook, 19 rules (rule 19 is the
  public-exposure guarantee).
- `references/interview.md`: question bank with skip logic.
- `references/brief-schema.md`: brief fields and validation checklist.
- `references/manifest-schema.md`: run manifest fields and manual procedures.
- `references/outline-spec.md`: how the visual outline is rendered.
- `references/webflow-conventions.md`: **the template onboarding fills in.**
  Sites, class naming, tokens, families, folders, slugs and SEO, schema, the
  instruction prefix, the source of truth and the mirror status, branching,
  breakpoints, Designer availability, access observed, rate limits, publish
  policy, reconciliation log.
- `references/catalog/README.md`: catalog entry format (including the `loose`
  and `candidate:` conventions and the `proposed` status) and the component
  metadata convention. `references/catalog/` itself ships empty; onboarding
  writes `catalog/<family>.md` into the source of truth (here, in the legacy
  git-checkout layout), and candidates land in `candidates/` beside it.
- `references/runs/README.md`: run records (brief, manifest, outline,
  snapshots, guard verdict per run). Ships empty. In the legacy git-checkout
  layout this is the working folder's `runs/`: live builds write here first,
  build Phase 0 checks it for an open manifest, and the maintain flow archives
  to it.
- `references/sync-state.json`: last-synced hashes per mirrored path; ships
  with `siteId` and `lastSync` null. A record: it lives in the source of truth,
  never in Webflow.
- `references/unsupported.md`: what the MCP cannot do, what depends on a site
  role, a plan, or a site limit, the Designer handoff for each, and the
  **access and entitlement table** every flow uses to classify a refused call.
- `references/examples/`: a filled catalog entry and a filled conventions file
  for a fictional site. Never read by a build; onboarding does not replace it.

Assets: `assets/brief.example.json` (a minimal brief that passes
`validate_brief.py`), `assets/outline.template.html` (template for
`render_outline.py`), and `assets/examples/` (the worked example's brief,
rendered outline, and dry-run manifest).

Codex metadata: `agents/openai.yaml`. Changes: `CHANGELOG.md`.
