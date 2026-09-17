# Unsupported through MCP, and the Designer handoff for each

What the Webflow MCP server cannot do, and what the report tells the human to
do instead. Every item the build flow meets from this list goes into the
brief's `unsupported[]` and the report's manual work list, worded as an
instruction someone can follow in the Designer.

Two kinds of row are mixed here and it matters which is which. Most rows are
**capability gaps**: the MCP server has no tool for the thing, on any site. The
rows at the end are **conditional**: whether they apply to you depends on the
Webflow site role or plan the connector user has, or on a limit your site may
or may not hit. `flows/onboard.md` measures those and records the answer in
`webflow-conventions.md`; do not carry a conditional row into a report without
checking it there first. A refused call (a 403, a 429, `ModeForbidden`) is
classified by the **access and entitlement table** at the end of this file;
every flow points there instead of carrying its own remedy.

| Gap | Kind | Why | Handoff wording (adapt the specifics) |
| --- | --- | --- | --- |
| **Webflow Interactions** (hover, scroll, load animations, carousels, accordions built with IX) | capability | No MCP tool creates or edits Interactions | "Open the page in the Designer, select `<element>`, add an Interaction: `<trigger>` → `<effect>` (`<duration>`, `<easing>`). The design showed `<what was observed>`." Never fake motion with embeds or scripts. |
| **Google or Adobe fonts** | capability | `data_fonts_tool` lists fonts; adding a Google or Adobe font is a site setting | "Site settings → Fonts → add `<family>` with weights `<list>`. Then map the `<type variable>` to it." Until then the template's font stays; report the substitution. |
| **Roles, access, and workspace settings** | capability | No MCP surface | "Ask a workspace admin to grant `<person>` `<role>` on `<site>`." |
| **Localization writes** | capability (deliberate) | Rule 16: primary locale only; `data_localization_tool` reads only for this skill | "Locale `<code>` needs the following strings translated after the page is final: `<slot list>`." |
| **Multipart asset upload on Claude.ai** | capability | `data_assets_tool > create_asset` then a POST to S3 needs sandbox network access; only used on Claude Code | "Upload `<file>` to Assets in the Designer (or give a public URL) and tell me the asset name; I will place it." |
| **Designer-dependent tools when the Designer is closed** (`element_snapshot_tool`, selection, `get_current_page`, `create_page_folder`, canvas navigation, branch context) | capability | Need the Designer open with the Bridge App running | "Open the site in the Designer with the Bridge App running and re-run verify; visual checks are pending." Folders are pre-created at onboarding so builds do not depend on this. |
| **Publishing** | capability (deliberate) | Rule 8 | "Review in the Designer, turn off the draft flag, publish yourself." |
| **Custom code beyond the approved outline** (page scripts, head code, embeds not in the outline) | capability (deliberate) | Rule 14 | "If `<embed>` is needed, add it in the Designer as an Embed element in `<section>`; the outline did not include it." |
| **CMS schema changes and Collection template pages** | capability (deliberate) | Not something this skill does | "This page needs a new CMS field `<name>` on `<collection>`; a maintainer adds it, then a later run binds it." |
| **Branch creation or merge before branch tooling is verified on your site** | conditional | Onboarding can verify only that element reads accept a branch page id; writes on a branch page and `create_branch` without the Designer stay unverified until the first branch-mode build (`webflow-conventions.md`, "Branching") | "Create the branch in the Designer (Pages → `<page>` → Create branch) and tell me the branch page ID" when the MCP path fails. |
| **Agent Instructions when the connector user's site role cannot read them** (HTTP 403 `forbidden`, "you cannot read this SiteAgentInstructions", on `search_instructions`, `read_instruction`, `create_instruction`) | conditional | A Webflow **site role** gate, not a missing tool and not an OAuth scope. Site managers and Designers read and manage instructions; Marketers and Content editors read them; Reviewers and Enterprise custom roles cannot read them. `webflow-conventions.md`, "Agent Instructions" and "Access observed", record whether it applies to you | The site-role row of the access table below, and the admin request in the README ("Asking your Webflow admin for access"). Until then the run keeps its brief and manifest as files, and onboarding hands over the catalog as a download bundle instead of installing it. |
| **Page-level JSON-LD when the connector user's role cannot write page settings** (HTTP 403 `insufficient_permissions` on `bulk_update_pages_schema_markup` while `query_pages_schema_markup` works) | conditional | A role permission on that resource, not a missing tool. Measured at onboarding; see "Schema defaults" | "Ask the Webflow workspace admin for a site role that can edit page settings, or paste this JSON-LD into Page settings > Custom code > Head for `<page>`: `<payload>`." |
| **Content inside loose sections and slot children on a site whose element reads exceed the request budget** | conditional | Webflow MCP prefetch, not a missing tool; `rules.md` rule 10. Whether it applies is the capability table in `webflow-conventions.md`, "Rate limits and response sizes". On a small site this row does not apply at all | "In the Designer set `<node>` in `<section>` to `<value>`" for each unreachable slot; the report lists them with the brief's final text so the edit is copy and paste. |
| **A Designer URL that opens a given page** | conditional | No MCP tool returns a page URL; guessed `https://<site-short-name>.design.webflow.com?pageId=` forms may open a tab the Bridge App is not bound to. `designer_tool > switch_page` navigates the connected tab | "In the Designer open Pages > `<folder>` > `<page title>` (page id `<id>`)." When the Bridge App is connected the run has already switched the canvas to the page. |
| **Tracking a CTA carries that the build did not write** (link extras, or the family's analytics shell component) | conditional | Not a missing tool: rule 14 keeps a build from writing a tracking scheme the site does not already use, and MCP cannot read click listeners in either direction. Whether it applies is `webflow-conventions.md`, "Site-level tracking" | "In the Designer open `<page>` and give `<CTA element>` the same link extras the site's other CTAs already carry: `<UTM keys or attribute name and value>`, as on `<sibling CTA>` on `<sibling page>`." For a missing shell: "place the existing `<component name>` component the `<family>` shell table names in `<section>`." Do not invent a new attribute, a new UTM set, or a new component. |

## How to phrase a handoff

One line each, imperative, with the element and the observed value. The reader
is a designer in the Webflow Designer, not an engineer. Example:

> Hero image: add a fade-in on page load (300 ms, ease-out) to the element
> `hero_media`. The design showed the image fading in from 0 to 100 percent.

## When something is not on this list

If a tool exists but fails for the run (rate limit, permissions, unexpected
error), that is not "unsupported": classify it by the table below, record the
failed step in the manifest, and retry or ask as the row says. Add a capability
row above only when the capability is missing from the MCP server itself; add a
conditional row only with the probe that established it and a pointer to where
the answer is recorded.

## Access and entitlement table

Every flow that meets a non-200 from the Webflow MCP server classifies it here
and repeats the row's exact ask. Nothing else in the skill carries a remedy for
a refused call. One probe per capability per conversation, never a retry loop:
a 403 is an answer, not a transient failure.

Read the error's `code` field, not the HTTP status: the first three rows are all
403s and they have different fixes. A message that names a scope is the OAuth
row. "You cannot read this SiteAgentInstructions" is the site-role row.
`insufficient_permissions` is the resource-permission row. The MCP server
enforces the connector user's Webflow permissions and roles, including custom
roles: an agent can do through it exactly what that user can do in the
Designer, and nothing more.

| Signal | Cause class | Who fixes it | Exact ask | Skill behavior |
| --- | --- | --- | --- | --- |
| HTTP 403, code `forbidden`, "User is not authorized to perform this action: you cannot read this SiteAgentInstructions" (the same class on create, update, and delete) | Webflow **site role**. Site managers and Designers read and manage Agent Instructions; Marketers and Content editors read them; Reviewers cannot read them, and access cannot currently be changed for Enterprise custom roles (on a custom role the separate "Use Webflow AI" permission also starts turned off) | a Webflow workspace admin or Site manager | "Assign the built-in **Designer** or **Site manager** site role to `<email>` on `<site>` (Site settings > Site access). If `<email>` is on a custom role, move them to a built-in role, or enable **Use Webflow AI** on the custom role and confirm Webflow AI is on for the workspace. An OAuth scope is not the problem and cannot fix this." | Guidance comes from the bundled copies or an attached catalog bundle; the Webflow instruction store is neither read nor written for the rest of the conversation and the report says so; the brief and the manifest stay as files or downloads. One probe, never retried. |
| HTTP 403, code `insufficient_permissions`, on `bulk_update_pages_schema_markup` while `query_pages_schema_markup` returns 200 | a role permission on that resource (page settings) | the same admin | "Give `<email>` a site role that can edit page settings on `<site>`." | The step is recorded `failed`; the JSON-LD payload is handed to the publisher for Page settings > Custom code > Head. |
| HTTP 403, code `missing_scopes`, "OAuthForbidden: You are missing the following scopes: ...". This is the only error that names an OAuth scope (for example `agent_instructions:read`), and it is what a scope problem looks like | the OAuth token the connector holds. Webflow's own OAuth app requests the scopes; an admin cannot grant them per user | the user | "Remove and re-add the Webflow connector (claude.ai: **+** > Connectors > Webflow) or re-authenticate the MCP server (Claude Code: `/mcp`), so the token is re-issued with the current scopes." | One probe, then continue degraded exactly as the site-role row; probe again only after the user says they reconnected. |
| HTTP 403, code `not_enterprise_plan_site`, on `list_branches` or `create_branch` | site plan: branching is an Enterprise feature | nobody in this session | none; the answer is final for the site | Branching is recorded unavailable in `webflow-conventions.md`, never offered, never retried. |
| `ModeForbidden` from a Designer tool | the Designer is in the wrong mode for the call | the user | "Switch the Designer to `<mode>` with the Bridge App connected and the tab in front, then tell me." | Retry once after the user confirms the switch; then record the step as a Designer handoff. |
| HTTP 429 | the per-minute request budget (rule 10) | nobody | none | The pacing rule from `webflow-conventions.md`, "Rate limits and response sizes": one ten-minute zero-traffic wait and one retry, then a `failed` manifest step naming the endpoint and a BLOCKED report. |
| Any other 4xx or 5xx | unclassified | ask | quote the status, the `code`, and the message verbatim | Stop the step, record it `failed` with the exact response, report. Do not retry blindly, and do not add a row here without the probe that established it. |

The MCP server has no whoami: it cannot report the connector user's site role.
Onboarding asks the user for it (Site settings > Site access shows it) and
records it in `webflow-conventions.md`, "Agent Instructions", next to the probe
result, so a later run can tell a role change from a Webflow change.
