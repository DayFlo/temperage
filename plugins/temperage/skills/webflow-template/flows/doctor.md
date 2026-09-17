# Flow: doctor ("check access")

Read-only. One probe per capability, each classified by the access and
entitlement table in `../references/unsupported.md`, and one read against the
configured store. It prints a capability matrix with the exact ask for each
refusal and the store decision it implies. `flows/onboard.md` step 0 and
`flows/build.md` Phase 0 run it; a user can ask for it alone ("check access")
at any time, and an administrator runs it after configuring the organization
(`README.md`, "Set up for an organization") to verify the setup.

Public users arrive with every role and plan combination. A deterministic
first run is what prevents a misdiagnosis: the 403 on Agent Instructions is a
Webflow site-role gate, not an OAuth scope, and only the error's `code` tells
the two apart.

References: `../references/unsupported.md` (the table), `../references/stores.md`
(discovery order, adapters), `../references/webflow-conventions.md` (where the
answers are recorded when onboarding runs this). Tools: `webflow_guide_tool`,
`data_sites_tool > list_sites`, `data_agent_instructions_tool >
search_instructions`, `data_pages_tool > list_pages`, `query_pages_schema_markup`,
`list_branches`, `designer_tool > get_current_page`, `get_all_breakpoints`, plus
one read through the store adapter.

## Rules

- **Once each, never a loop.** Every probe runs exactly once. A 403 is an
  answer. A 429 ends the probing for this conversation and the report says
  which probes were not reached.
- **Read-only.** Nothing is created, updated, or deleted in Webflow or in any
  store. The page schema *write* permission cannot be probed without writing,
  so it is reported from `webflow-conventions.md` ("Schema defaults", measured
  at onboarding) or as "unknown until the first build writes".
- **Serialized.** One call at a time, in the order below, with the gap
  `webflow-conventions.md` prescribes when it is filled; on a site that has not
  been onboarded, a few seconds between calls.
- **Quote, do not paraphrase.** For each refusal record the HTTP status, the
  error `code`, and the message verbatim; classification comes from the table,
  not from the wording of the message alone.

## Probes, in order

| # | Probe | 200 means | Refusal classes to expect | Store consequence |
| --- | --- | --- | --- | --- |
| 1 | `webflow_guide_tool` | the MCP server answers; record the version string it returns and compare it with `testedMcpVersion` in `org.json` or "Tested MCP version" in the conventions file | none; a failure here means the connector is not connected | none |
| 2 | `list_sites` | the connector is authorized; the user picks the site | `missing_scopes` (OAuth row) | none |
| 3 | `search_instructions`, no filter | the role can **read** Agent Instructions; if `rules/<prefix>.md` is among the hits, read it and adopt its pointer block (`stores.md` section 5) | 403 `forbidden` "you cannot read this SiteAgentInstructions" (site-role row: Reviewer or a custom role) | read refused: guidance comes from the source of truth or a bundle; the mirror is skipped for the conversation. Whether the role can also **write** is not probed here; the built-in role the user names in onboarding step 1 answers it (Designer and Site manager write; Marketer and Content editor read only) |
| 4 | `list_pages` (first page of results) | pages are readable | `missing_scopes` | none |
| 5 | `query_pages_schema_markup` on one page from probe 4 | page schema is readable | `insufficient_permissions` (resource-permission row) | the write side is reported from the conventions file, never probed |
| 6 | `list_branches` | branching is available on the plan | 403 `not_enterprise_plan_site` (plan row; final for the site) | none; `branch` isolation is offered only on 200 |
| 7 | `designer_tool > get_current_page` | the Designer is open with the Bridge App in front | a failure is not a refusal: the response carries the Bridge App launch link; run the foreground procedure from `build.md` Phase 0 step 5 **once**, never store the link | none |
| 8 | `get_all_breakpoints` (only after a 7 that succeeded) | the breakpoint list; compare with the conventions file and say so if it changed | `ModeForbidden` (Designer-mode row) | none |
| 9 | Store read: `list` on the configured source of truth (`stores.md` section 3), or, when none is configured, name the document connectors present in the conversation's tool list | the store is reachable for this account | a connector 401 or 403, a missing connector, a working folder that is not writable | reachable: records and guidance go there. Not reachable: the working folder (Cowork, Claude Code) or downloads (claude.ai), and the report names the maintainer who can file them |

## The matrix

Print one table, in this shape, then stop:

| Capability | Result | Classification | Exact ask | What the skill does |
| --- | --- | --- | --- | --- |
| Webflow MCP version | `<version>`; tested `<version or unset>` | drift or none | none | says so once when they differ |
| Sites | `<n>` visible; using `<site>` (`<id>`) | | | |
| Agent Instructions read | 200 / 403 `<code>` | `<table row>` | `<row's exact ask>`, with the README's admin request when it is the site-role row | mirror on / mirror skipped |
| Agent Instructions write | `<Designer or Site manager: yes / Marketer or Content editor: no / unknown>` (from the role the user named) | | | guidance mirrored / read-only mirror |
| Pointer block | adopted `<sourceOfTruth>` at `<location>` / none / disagrees with the administrator configuration | | | |
| Page schema read | 200 / 403 | | | |
| Page schema write | from conventions: allowed / refused / unmeasured | | | JSON-LD written / handed to the publisher |
| Branching | available / `not_enterprise_plan_site` | | | `branch` offered / never offered |
| Designer | reachable after `<n>` attempts / unreachable | | | snapshots and folder creation on / off |
| Breakpoints | `<list>` / unchanged / changed since onboarding | | | |
| Source of truth | `<adapter>` at `<location>`: reachable / not reachable / not configured | | | where records and guidance go |
| Working folder | `<path>`: writable / not set / not applicable on claude.ai | | | write-ahead layer on / downloads |

Two closing lines: the store decision in one sentence ("Guidance from
Confluence, mirrored to Webflow; records to Confluence with the working folder
as write-ahead"), and the single most useful next step (usually the exact ask of
the first refused row, or "nothing refused; run onboarding" on a site with no
pointer).

When onboarding runs this flow, every row lands in `webflow-conventions.md`:
the probe results in "Agent Instructions", "Access observed", "Branching",
"Designer availability", and the reconciliation log, each with the date.
