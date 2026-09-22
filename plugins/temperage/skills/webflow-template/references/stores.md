# Stores: where guidance and records live

The skill keeps two kinds of data in two kinds of place. This file is the
specification every flow reads when it stores or fetches anything: the data
kinds, the one layout, the adapters, the organization configuration, the
Webflow pointer, the discovery order, and the failure rules. Nothing else in
the skill defines a store.

## 1. Data kinds

| Kind | Examples | Properties | Source of truth | Webflow mirror | Working folder |
| --- | --- | --- | --- | --- | --- |
| **Guidance** | the rulebook, `catalog/<family>.md`, `catalog/index.md`, `conventions.md` | small, read at every build, written rarely, reviewed | yes | **yes**, whenever the connector user's site role allows | cache |
| **Records** | `runs/<date>-<slug>.*`, `candidates/<slug>.md`, `sync-state.json` | write-heavy, must survive a 429, a 403, and a closed tab; audit only | yes | **never** | write-ahead, always first |
| **Cache** | the site inventory (`site-inventory.json`, `.md`, `.index.json`) | large, regenerable by re-running `flows/onboard.md` step 3 | no | never | only here, never shared |

Guidance is what an agent reads before it acts. Webflow Agent Instructions are
the one store every MCP client on the site sees, and Shared Library can push
them across sites, so guidance is **mirrored** there. Records are not guidance:
they inflate every agent's instruction discovery on the site, whether drafts
are hidden from other agents is unverified, the store has no readable history
or diff, and writes need the Designer or Site manager role. Records never go
to Webflow. The rulebook itself ships in the skill (`rules.md`); the mirror
copy at `rules/<prefix>.md` is what other agents read.

## 2. One layout on every store

Every store, page-based or file-based, holds the same tree. A path below is
the key an adapter reads and writes; on a page store it is the page title.

```
webflow-template/
  org.json                          store pointer, prefix, tested MCP version (section 4)
  sites/<shortName>/
    conventions.md                  the filled webflow-conventions.md
    catalog/<family>.md             one entry per family
    catalog/index.md                the catalog index (flows/sync.md defines the body)
    catalog/candidates/README.md    optional; the candidates process
    runs/<date>-<slug>.brief.json
    runs/<date>-<slug>.manifest.json
    runs/<date>-<slug>.outline.html
    runs/<date>-<slug>.pre.snapshot.json
    runs/<date>-<slug>.post.snapshot.json
    runs/<date>-<slug>.guard.verdict.json
    candidates/<slug>.md
    sync-state.json
```

`<shortName>` is the site's short name as `list_sites` returns it. The Webflow
mirror uses the existing instruction paths and nothing else: `rules/<prefix>.md`,
`<prefix>/SKILL.md` (the index), `<prefix>/conventions.md`,
`<prefix>/catalog/<family>.md`.

**Legacy layout.** A git checkout that carried a catalog before 1.1.0 keeps it
where it was: `references/catalog/`, `references/runs/`,
`references/webflow-conventions.md`, `references/sync-state.json` inside the
skill folder stand for `sites/<shortName>/catalog/`, `runs/`, `conventions.md`,
`sync-state.json`. The flows accept either shape; a new working folder uses the
layout above and is never inside the plugin directory, which is read-only and
replaced on update.

## 3. Adapters

One interface, three operations: `list(prefix)`, `read(path)`, and
`write(path, body)`. Every body is the verbatim markdown or JSON file; the path
is the layout path above. Tool names differ between the claude.ai connectors,
Cowork, and a Claude Code MCP server; use whichever tool performs the operation
named.

| Adapter | Location config | Mapping | History | Notes |
| --- | --- | --- | --- | --- |
| **Notion** | parent page id | one child page per path, title = the path; the verbatim file **attached** as a file upload (`create file upload`, then attach it to the page); page body = the human summary and the attachment. `read` = download the attachment; `write` = a new upload attached to the same page | page history, retention depends on the plan | the attachment is the truth, not the page body. Verified 2026-09-16 against the Notion API request limits: a rich text `text.content` is capped at 2000 characters and any block array at 100 elements, so a code block would need chunking and reassembly; the attachment is byte-exact. The workspace's `max_file_upload_size_in_bytes` bounds a single write; read it before the first write and apply the size rule below |
| **Working folder** | an absolute folder path chosen by the user | files exactly as laid out | none by itself; a Drive or OneDrive desktop-synced folder adds versions, and a **git checkout** adds a pull request, the only path that has one | Cowork and Claude Code (Codex too). Never a path inside the plugin directory. Always the write-ahead layer (section 7), whatever the source of truth |
| **Downloads** | none | one JSON bundle and one self-contained HTML page at the end of every phase (the bundle format is below) | the user keeps the files | claude.ai with no document connector, and any submitter whose account cannot reach the source of truth. Never a zip: chat accepts HTML, JSON, and plain text |
| **Webflow mirror** | the instruction prefix | guidance paths only, per-path confirmation, `create_instruction` when absent and `update_instruction` when present, the three-way hash rule from `flows/sync.md` | none readable | written only when the connector user's site role allows (`unsupported.md`, access and entitlement table); never a home for a record |

**Page stores.** Notion is the only page-store adapter. The skill detects the
Notion connector from the tools available in the conversation and, when no
configuration already answers, confirms it as the source of record in one
sentence and stores the answer (section 4). Confluence is not an adapter: it
was dropped on 2026-09-22 to keep one page-store mapping to maintain and test.
Google Drive is not an adapter: its
connector creates files and folders but cannot edit an existing file. Slack
canvases are not an adapter: no structure, weak history. Webflow CMS is not an
adapter: CMS items have staged and live states and are publish-shaped by design
(`rules.md` rule 19), so records kept there would sit one publish away from the
public internet; there is no "just for drafts" exception.

**Size rule.** Measure every body before a write. A body over about 200 KB (a
site inventory always; a pre or post snapshot on a large site) is never written
to a page store or to the Webflow mirror: it stays in the working folder or in
the downloads, and the manifest records its sha256 and where it is. The 256 KB
ceiling per Agent Instruction is unchanged. Nothing is ever silently truncated.

**The bundle.** `<prefix>-catalog-bundle.json` is one JSON object:
`schema` (2), `prefix`, `siteId`, `shortName`, `generatedAt` (ISO 8601),
`sourceOfTruth` and `location` (as in `org.json`, so a reader knows where the
live copy is), `rulebook`, `conventions`, `index` (markdown strings),
`families` (one entry each: `slug`, `status`, `version`, `body`), and, new in
1.1.0, `runs` (one entry per run: `runId`, `brief`, `manifest`, `guardVerdict`
as JSON objects, `outline` as an HTML string; snapshots only when under the
size rule) and `candidates` (`slug`, `body`). The same content as one
self-contained HTML page when the user would rather read it than store it.

## 4. Organization configuration: `org.json`

`references/org.json` ships with every value empty, and `checks/repo_check.py`
fails if the public plugin carries any organization's values. An
administrator fills it once; nobody else sees a setup question. Every value is
non-sensitive, so both `${user_config.*}` substitution and plain file reading
work, and no token is ever in it: the connectors own authentication.

```json
{
  "schema": 1,
  "sourceOfTruth": "",
  "location": {
    "notionParentPageId": "",
    "workingFolder": ""
  },
  "instructionPrefix": "page-templates",
  "allowedStores": [],
  "sites": [],
  "testedMcpVersion": "",
  "configuredBy": "",
  "configuredOn": ""
}
```

| Key | Values | Meaning |
| --- | --- | --- |
| `sourceOfTruth` | `notion`, `working-folder`, `downloads` | The adapter guidance and records are written to and read from first |
| `location` | the keys the chosen adapter needs; the rest stay empty | Notion: `notionParentPageId`. Working folder: `workingFolder`, an absolute path, or empty to ask once per machine |
| `instructionPrefix` | default `page-templates` | The Webflow instruction prefix; the same value `webflow-conventions.md`, "Toolkit settings", records per site |
| `allowedStores` | a subset of the three values, or empty | What a Webflow pointer (section 5) may select; empty means unconstrained. A pointer naming a store outside this list is shown to the user and not adopted |
| `sites` | Webflow site ids, or empty | Sites this configuration applies to; empty means any site |
| `testedMcpVersion` | the version string `webflow_guide_tool` returned when the configuration was last verified | Preflight compares it with the live value and says so once when they differ |
| `configuredBy`, `configuredOn` | a name or role, an ISO date | Who to ask, and how old it is |

**Three ways the configuration reaches a user.**

1. **claude.ai organization skill** (Team, Enterprise). An owner uploads the
   skill zip with `references/org.json` filled at Organization settings >
   Skills. It is enabled for everyone, recipients cannot edit it, and a
   re-upload updates everyone at next use. Onboarding's "configure for
   organization" path emits that zip (`flows/onboard.md` step 9).
2. **Claude Code and Cowork plugin.** `plugin.json` declares `userConfig`
   fields with the same meanings (`source_of_truth`,
   `notion_parent_page_id`, `working_folder`,
   `instruction_prefix`, `allowed_stores`, `sites`, `tested_mcp_version`).
   The user answers once when enabling the plugin, or an administrator presets
   them in managed settings under `pluginConfigs["temperage"].options`, which
   users cannot override. The values reach this file as substitutions:

   | Field | Value in this conversation |
   | --- | --- |
   | source of truth | `${user_config.source_of_truth}` |
   | Notion parent page id | `${user_config.notion_parent_page_id}` |
   | working folder | `${user_config.working_folder}` |
   | instruction prefix | `${user_config.instruction_prefix}` |
   | allowed stores, sites | `${user_config.allowed_stores}`, `${user_config.sites}` |
   | tested MCP version | `${user_config.tested_mcp_version}` |

   A cell that still reads as the literal `${user_config.…}` placeholder, or is
   empty, means the surface did not substitute it (claude.ai, Codex) or the
   user left it blank: treat it as unset and fall through the discovery order.
   Cowork does not read Claude Code's managed settings; the Cowork path for a
   preset is an organization-hosted copy of the plugin with `org.json` filled.
3. **The Webflow pointer** (section 5), which reaches every agent on the site
   whatever client installed the skill.

## 5. The Webflow pointer block

The first fenced block of `rules/<prefix>.md`, written by onboarding when the
connector user's role allows and pushed across sites with Shared Library:

````
```webflow-template
sourceOfTruth: notion
location: NOTION_PARENT=<page id>
instructionPrefix: page-templates
updated: <ISO date>
```
````

`location` is one line: `NOTION_PARENT=…` for Notion, `FOLDER=…` for a working folder, empty for downloads. Anyone whose
role can read Agent Instructions gets the store location from the site itself.
The block carries a location, never a credential, so a Designer who edits it
can at worst redirect where records land, and `allowedStores` in `org.json`
bounds even that. The rest of `rules/<prefix>.md` is the rulebook, verbatim;
the pointer is part of the hashed body like everything else the sync flow
pushes.

## 6. Discovery order and precedence

Run once per conversation, at preflight (`SKILL.md`), by `flows/doctor.md`,
and by `flows/onboard.md` step 0. Webflow goes first because the configuration
is site-scoped, the probe is already part of preflight, the rulebook binds
every agent on the site, and the Webflow copy is live while skill copies and
caches go stale.

```
0  Pick the site                 list_sites; org.json may name a default
1  Webflow                       search_instructions, one call, no filter
   ├─ 200 with a pointer block .. adopt it (site truth: WHERE)
   ├─ 200, no pointer ........... not onboarded, or no pointer yet; continue
   └─ non-200 ................... classify by the access table; say so; continue; never retry
2  Administrator configuration   ${user_config.*} or references/org.json in the skill copy
   └─ supplies defaults and constraints (WHAT IS ALLOWED); when it disagrees with 1, show both and ask
3  Working-folder org.json       the cache an earlier run wrote (Cowork, Claude Code)
4  Ask once                      which document connector is connected? working folder? downloads?
   └─ write the answer to 3, and the pointer to Webflow when the role allows
```

**Read order for guidance**, once the source of truth is known: the source of
truth through its adapter; else the Webflow mirror (say "the mirror may lag
the source of truth"); else an attached bundle (say the `generatedAt` it
carries); else stop at template selection and point at `flows/onboard.md`.

**Read order for records** (an open manifest, a candidate): the working folder;
then the source of truth's `runs/` and `candidates/`; then ask for the download.
Never Agent Instructions.

## 7. Failure handling and concurrency

- **Write-ahead.** A record is written to the working folder (Cowork, Claude
  Code, Codex) or handed over as a download at the end of the phase (claude.ai)
  **before** any network store, and the manifest is appended before every
  Webflow write (`rules.md` rule 9). The local layer is the only one that never
  fails on the network.
- **A failed store write** (a page store 4xx or 5xx, a connector timeout) is
  recorded in the manifest as a `failed` step naming the adapter and the path,
  retried **once** at the end of the next phase, and otherwise left for the
  report; the run continues. A failed mirror write follows the access table:
  a 403 stops mirroring for the conversation, is said once, and is never
  retried.
- **A submitter without the store connector** reads guidance from the Webflow
  mirror, else from an attached bundle; their records go to downloads, and the
  report names a maintainer who can file them into the source of truth.
- **Concurrency.** Run ids (`<date>-<slug>`) are unique, so parallel builds on
  one site do not collide on `runs/`. Guidance writes are rare and confirmed;
  the three-way hash compare in `sync-state.json` (`flows/sync.md`) detects a
  concurrent edit and shows both versions instead of overwriting either.
- **Limits.** 256 KB per Agent Instruction (unchanged). Notion: 2000
  characters per rich text `text.content`, 100 elements per block array, a
  per-workspace file upload size (verified 2026-09-16); the attachment mapping
  is why. The size rule in section 3 keeps bodies under about 200 KB on the
  page store.
- **Never.** No record in Agent Instructions. No store inside the plugin
  directory. No token in `org.json`, in a pointer, or in a bundle. No CMS
  collection as a store.
