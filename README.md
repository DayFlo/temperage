# temperage

Turns a design into an **unpublished draft page** in a Webflow site, built
from a reusable template family: the components, styles, variables, and
layout conventions the site already has. Never publishes.

This repository is a plugin marketplace. One plugin,
`temperage`, carries the `webflow-template` skill. Claude Code, Cowork, and
Codex install from the repo. Claude.ai does not: zip the skill folder and
upload it. After that, the catalog lives in your team's Notion (or a working
folder), and is mirrored into Webflow Agent Instructions so
every agent on the site reads the same rules.

**Onboard first.** Families name components that exist on one site, so
there is no portable catalog to ship. Until onboarding has run, a build
stops at template selection.

Every build: receive a design → digest it → interview the submitter →
pick a family → approve a visual outline → build an unpublished page →
verify by readback → report and hand off.

## Install

### Claude Code

```
/plugin marketplace add DayFlo/webflow-template-skill
/plugin install temperage@webflow-template-skill
```

Invoke: `/temperage:webflow-template`. Plugin skills are
namespaced. The unqualified form works only for a copy in
`.claude/skills`. The skill will not auto-fire
(`disable-model-invocation: true`).

The plugin ships `.mcp.json` pointing at `https://mcp.webflow.com/mcp`.
Claude Code prompts for the Webflow OAuth flow on first use. Enabling the
plugin also prompts for the store configuration (source of truth, its
location, the instruction prefix); leave it blank to be asked once, or to
follow the pointer the site already carries. An administrator presets the
same values in managed settings under `pluginConfigs` (see "Set up for an
organization").

### Cowork

In the Claude desktop app open the **Cowork** tab, then **Customize →
Plugins → Add from a repository**, and paste
`https://github.com/DayFlo/webflow-template-skill`. Install **temperage**.
Invoke the skill by its namespaced name, `temperage:webflow-template`; it is
not listed among the available skills because it never auto-fires.

What a Cowork run on 2026-09-22 showed:

- Webflow access comes from the **claude.ai Webflow connector**. The plugin's
  `.mcp.json` server did not load, so connect Webflow under claude.ai
  Connectors first.
- The session runs in a **cloud container** with Python 3.11, so the helper
  scripts run. `~` there is `/root`, not your Mac. Records written to a
  container folder last only for the session; connect a folder from your Mac
  if you want a working folder that persists, or use Notion as the source of
  truth.
- Hooks and sub-agents are Cowork-only features; this plugin ships neither.

### Codex

Copy or symlink the skill folder into the Codex user skills directory:

```
ln -s "$PWD/plugins/temperage/skills/webflow-template" "$HOME/.agents/skills/webflow-template"
```

Register the MCP server in `~/.codex/config.toml`:

```toml
[mcp_servers.webflow]
url = "https://mcp.webflow.com/mcp"
```

Invoke: `$webflow-template`. `agents/openai.yaml` sets
`allow_implicit_invocation: false`.

### Claude.ai

Zip the skill folder (not the marketplace root) and upload it, or have a
Team / Enterprise owner provision it.

```
cd plugins/temperage/skills
zip -r webflow-template.zip webflow-template
```

The archive must contain the `webflow-template/` directory as its root.
Claude.ai rejects a zip whose folder name does not match the skill name.
Do not send the GitHub URL; a private repository will not help.

1. Paid Claude (Pro, Max, Team, or Enterprise). Turn on **Code execution
   and file creation** (Pro / Max: **Settings → Capabilities**; Team /
   Enterprise: **Organization settings → Skills**).
2. **Customize → Skills** → **+ Create skill** → **Upload a skill**, or
   have an owner add it under **Organization settings → Skills → + Add**.
3. Connect the Webflow connector in a chat (**+** → **Connectors** →
   **Webflow**). Your Webflow **site role** decides whether the catalog can
   live in Webflow Agent Instructions; see Requirements.
4. Start a chat and select **webflow-template** by name.

Python scripts only run there if code execution is enabled. Every script
has a written fallback the skill follows by hand when it is not.

## Usage

Ask for a flow by name: **onboard**, **check access**, **build**,
**maintain**, **sync**, or **resume**.

| Surface | Invoke |
| --- | --- |
| Claude Code | `/temperage:webflow-template` |
| Cowork | the skill by name, or `/temperage:webflow-template` |
| Codex | `$webflow-template` |
| Claude.ai | select **webflow-template** by name |

## Set up for an organization (admin, once)

The skill separates **where the catalog lives** (the source of truth) from
**the Webflow mirror** every agent on the site reads. An admin configures
both once; nobody else sees a setup question.

1. **Pick the source of truth.** Notion, through the Claude Notion
   connector. Create an empty parent page for it (for example `Webflow templates`) and share it with everyone
   who will run builds. If your team has neither, the skill falls back to a
   working folder (Cowork, Claude Code) or downloads (claude.ai).
2. **Run the onboard flow with "configure for organization".** Do this from
   an account whose Webflow site role is Designer or Site manager. Onboarding
   measures the site, writes the catalog to the store you picked, writes the
   rulebook and a **pointer** to the store into Webflow Agent Instructions
   (`rules/page-templates.md`), and hands you two files: the catalog bundle
   and an **organization zip** of the skill with `references/org.json` filled
   in.
3. **Provision the skill.** Team or Enterprise owner: **Organization settings
   → Skills → + Add** and upload the organization zip. It is enabled for
   everyone; members cannot edit it; re-upload to change the configuration.
   Cowork or Claude Code plugin users instead answer the plugin's
   configuration prompt once, or an administrator presets the same values in
   managed settings under `pluginConfigs`:

   ```json
   {
     "pluginConfigs": {
       "temperage": {
         "options": {
           "source_of_truth": "notion",
           "notion_parent_page_id": "<parent page id>",
           "instruction_prefix": "page-templates"
         }
       }
     }
   }
   ```
4. **Optional: push across sites.** Agent Instructions are a Shared Library
   resource. Add `rules/page-templates.md` to your library and install it on
   the other sites in the workspace so the pointer travels with them.
5. **Verify.** Ask the skill to **check access**. It probes Webflow once per
   capability and prints what is readable, what is writable, and the exact
   ask for anything refused.

**Per user, afterwards.** Connect the Webflow connector, plus the Notion
connector if that is the store. Nothing else.

## Onboarding

`flows/onboard.md` reads a site once, read-only, and writes the catalog,
filled conventions, a local site inventory, and the rulebook installed into
Webflow Agent Instructions. Its first step finds the store (the site's
pointer, then the organization configuration, then it asks once):

| Store | Holds | Reviewed by | Who can write |
| --- | --- | --- | --- |
| **Notion** (source of truth) | catalog, conventions, index, run records, candidates | page history and comments | anyone with connector access to the parent page |
| **Working folder** (Cowork, Claude Code) | the same, as files; a git checkout gets a pull request | the folder's own history, or nobody | the user |
| **Downloads only** (claude.ai without a connector) | the same, as one JSON bundle and one HTML page | nobody | the user keeps the files |
| **Webflow Agent Instructions** (mirror, always attempted) | rulebook, index, conventions, catalog entries; never records | per-path confirmation | Designer or Site manager site role |

Measuring is identical on every store. Every path also hands over a
**download bundle** (one JSON file, or one HTML page). A build accepts
that bundle when it cannot read the store. The working folder is also the
write-ahead layer: a record is written there before any network store, so it
survives a rate limit, a refusal, or a closed tab.

A page store has no pull request. What stands in for one: the page's version
history, a confirmation before each guidance write, proposed families kept as
drafts until a maintainer confirms them, and the bundle as a portable copy.

A worked example for a fictional site lives under
`plugins/temperage/skills/webflow-template/references/examples/`
and `.../assets/examples/`. Onboarding does not touch it.

## What this can and cannot make public

Using this skill must not put anything on the public internet. Only one of
the writes it makes can, and it says so at the moment it makes it. The
full rule is `references/rules.md` rule 19; the same table is in
`SKILL.md`.

| What | Public? | The guarantee |
| --- | --- | --- |
| **The draft page** | No | Created with `draft: true` set explicitly and confirmed by readback. Draft pages are excluded from publishing, so the page does not go live at the next site publish either. A human turns the flag off. |
| **New components, styles, variables** | Not yet - **at the next site publish, yes** | Site-level, so they ship whenever anyone next publishes the site, even though the page stays a draft. Every run reports them as the ships-at-next-publish list. Branch mode keeps them off main until merge. |
| **Uploaded assets** | **Yes, immediately** | This is the one exception. `asset_tool > upload_image_by_url` puts the file in the site's asset library, and Webflow serves library assets from a public CDN URL from the moment of upload, before any publish and whether or not the page is ever published. The build warns and asks before every upload, prefers an asset already in the library, and reports each one as an "already public" line, separate from and more urgent than ships-at-next-publish. Deleting an asset later does not un-serve a URL someone already has. |
| **Branch staging publish** | Gated, not open | Only on explicit request in that turn, only in branch mode, only to staging, never production. Measured, not assumed: an anonymous request to a Webflow branch staging URL redirects to the Webflow login and returns HTTP 403. |
| **Agent Instructions** (the guidance mirror; never a record) | Evidence says no; no vendor statement | Gated by Webflow site role (Site manager and Designer manage; Marketer and Content editor read; Reviewer and custom roles cannot read), delivered to authorized MCP clients as site metadata, no publish path, never in page content. Not a guarantee: onboarding runs a one-time check per site (write a throwaway instruction with a unique marker, have a human publish on their normal cadence, confirm the marker appears nowhere in public output, delete it). |
| **CMS items** | Never used | Standing non-goal. The Data API models CMS items with staged and live states and publish and unpublish events: they are publish-shaped by design, so a catalog entry, brief, candidate, or run record kept in a collection would sit one publish away from the public internet. |

`checks/repo-check.sh` fails if this section disappears from either file,
or if the rulebook stops forbidding `publish_site`.

## Safety

The table above is the publish story. These are the rest:

- Never calls `publish_site`. `publish_branch` only to staging, only in
  branch mode, only on explicit request, always recorded.
- Never edits a pre-existing shared definition (class, base variant,
  variable). A pre/post inventory guard fails the run if anything
  pre-existing changed.
- Never deletes anything the run did not create.
- Every Webflow write is appended to a run manifest before the next
  write.
- The Webflow Designer Bridge App launch link is a credential. Never
  written to a file, a manifest, a run record, or a commit.

The full rulebook is
`plugins/temperage/skills/webflow-template/references/rules.md`
(19 rules). It is installed into the site's Agent Instructions so every
other agent connected to the site reads the same rules.

## Requirements

- Claude Code, Codex, or paid Claude.ai (Pro, Max, Team, or Enterprise)
- Webflow MCP connector (OAuth). Claude Code prompts on first use.
- Python 3.11 for local scripts (standard library only). Claude.ai uses a
  written fallback when code execution is off.

**Webflow site role.** The Webflow MCP server enforces your site role: an
agent can do through it exactly what you can do in the Designer. To *read*
Agent Instructions you need the built-in Marketer, Content editor, Designer,
or Site manager role; to *write* them (onboarding's install step, sync) you
need Designer or Site manager. Reviewers and Enterprise custom roles cannot
read Agent Instructions at all, and that access cannot currently be changed
for custom roles; on a custom role the separate "Use Webflow AI" permission
also starts turned off. The symptom is HTTP 403 `forbidden`, "you cannot read
this SiteAgentInstructions". It is not an OAuth scope problem: a scope error
has the code `missing_scopes` and names the scope, and no admin can grant
scopes per user. Everything else in the skill works with any role that can
edit pages and components; without instruction access the skill reads its
catalog from an attached bundle and says so. The full classification of every
refused call is the access and entitlement table in
`plugins/temperage/skills/webflow-template/references/unsupported.md`.

## Troubleshooting

Every refused call is classified by the access and entitlement table in
`references/unsupported.md`. The short version:

| You see | It means | Who fixes it, and how |
| --- | --- | --- |
| 403 `forbidden`, "you cannot read this SiteAgentInstructions" | your Webflow site role cannot read Agent Instructions (Reviewer, or a custom role) | a Webflow workspace admin assigns the built-in Designer or Site manager role (Site settings > Site access); use the template below |
| The catalog was written but other agents on the site cannot see it | your role reads instructions but cannot write them (Marketer, Content editor), so the mirror was skipped | a Designer or Site manager runs the sync flow once |
| 403 `insufficient_permissions` on page schema | your role can read page schema but not edit page settings | the same admin; meanwhile the JSON-LD is handed to the publisher |
| 403 `missing_scopes`, "OAuthForbidden: You are missing the following scopes" | the connector's OAuth token is stale; this is the only error that is about scopes | you: remove and re-add the Webflow connector, or re-authenticate the MCP server |
| 403 `not_enterprise_plan_site` | branching is an Enterprise feature | nobody; branch mode is never offered on this site |
| `ModeForbidden` | the Designer is in the wrong mode | you: switch the Designer mode with the Bridge App connected |
| 429 | the per-minute request budget | nobody; the skill waits and retries once, as onboarding measured for your site |
| **"I cannot read the catalog for this site."** | the store is unreachable for your account and no bundle is attached | attach the catalog bundle from onboarding, or ask a maintainer to run onboarding, or ask your Webflow admin for the role above |

## Asking your Webflow admin for access

Copy, fill in, send:

> I need read and write access to AI Agent Instructions on the site
> `<site name>` (site ID `<id>`) for `<email>`. The Webflow MCP server returns
> HTTP 403 "you cannot read this SiteAgentInstructions" for my account. Per
> Webflow's Help Center, only the built-in Site manager and Designer roles can
> read and manage Agent Instructions, and access cannot currently be changed
> for custom roles. Please either set my Site role to Designer (Site settings >
> Site access), or, if I am on a custom role, switch me to the built-in
> Designer role or enable the "Use Webflow AI" permission on my role, and
> confirm Webflow AI is enabled for the workspace. I will verify by opening the
> Instructions panel in the Designer and re-running the read.

## Development

```
.claude-plugin/marketplace.json        Claude Code marketplace manifest
.agents/plugins/marketplace.json       Codex marketplace manifest
checks/repo-check.sh                   structure + disclosure checks
plugins/temperage/
  .claude-plugin/plugin.json
  .mcp.json                            Webflow MCP server
  skills/webflow-template/
    SKILL.md                           entry point, explicit invocation only
    flows/                             onboard, doctor, build, maintain, sync, resume
    references/                        rulebook, stores spec, org.json, schemas, catalog format
    scripts/                           six Python 3 scripts, standard library only
```

```
cd plugins/temperage/skills/webflow-template/scripts
python3 -m unittest discover -s tests
```

```
bash checks/repo-check.sh
REPO_CHECK_LIVE=1 bash checks/repo-check.sh   # also runs claude plugin validate
```

120 unit tests. No live Webflow run in this repository, and nothing here
talks to the network.

Every Webflow-shaped identifier in this tree is synthetic:

| Shape | Pattern | Used for |
| --- | --- | --- |
| 24-hex | `0000000000000000000000xx` | sites, pages, folders |
| UUID | `1111aaaa-0000-4000-8000-000000000xxx` | components, props, variables |

`checks/repo-check.sh` fails on any identifier outside those two
patterns, on any Bridge App launch link, and on any other long
secret-shaped token.

## License

MIT. See `LICENSE`. Copyright (c) 2026 Dayton Floyd.
