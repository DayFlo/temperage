# Changelog

All notable changes to the `webflow-template` skill. The format follows
Keep a Changelog; versions follow semver and are mirrored in `SKILL.md`
frontmatter (`metadata.version`).

## [1.1.0] - 2026-09-16

### Changed

- **Storage model.** Guidance (the rulebook, the catalog entries, the index,
  the conventions) and records (briefs, run manifests, snapshots, guard
  verdicts, candidates, sync state) now have different homes. Guidance lives
  in a **source of truth** and is **mirrored** into Webflow Agent Instructions
  under `<prefix>` whenever the connector user's site role allows. Records
  live in the source of truth only and are **never** written to Agent
  Instructions; the build flow no longer calls `create_instruction` for a run
  record or a candidate. `references/stores.md` is the specification.
- The source of truth is one of **Notion** (detected from the connected
  connectors and confirmed once), a **working folder** (Cowork, Claude Code, Codex; a git
  checkout is the only path with a pull request), or **downloads only**
  (claude.ai with no document connector). "Repository" and "Webflow" are no
  longer store choices: a git checkout is a working folder, and Webflow is the
  mirror in every configuration. A pre-1.1.0 catalog under the skill's
  `references/` keeps working as the legacy layout.
- The **working folder is the write-ahead layer**: the manifest and the brief
  are written there (or handed over as downloads at the end of every phase on
  claude.ai) before any network store, so a run record survives a 429, a 403,
  or a closed tab.
- `flows/onboard.md` step 0 runs the discovery order (Webflow pointer,
  administrator configuration, cached `org.json`, ask once) before asking
  anything; step 8 writes the source of truth first, then the four guidance
  paths of the mirror, and nothing else; step 9 can emit the organization zip;
  step 11 reports store by store.
- `flows/sync.md` pushes source of truth to mirror and pulls guidance edits
  made in the Webflow Instructions panel back; candidates and run records are
  no longer pulled because they are no longer written there. The toolkit-owned
  paths are the four guidance paths. `rules/<prefix>.md` now opens with the
  pointer block.
- `flows/build.md` Phase 0 reads guidance from the source of truth, else the
  mirror (warning that it may lag), else an attached bundle; looks for an open
  manifest in the working folder and the source of truth; Phase 5 writes
  records locally first; Phase 7 names the store each artifact went to and the
  mirror status.
- `flows/resume.md` locates the manifest in the working folder, then the
  source of truth, then asks for the download. `flows/maintain.md`
  "Maintaining without a repository" is now "Maintaining in a page store", and
  Housekeeping moves pre-1.1.0 records out of Agent Instructions.
- `references/webflow-conventions.md` "Toolkit settings" gains Source of
  truth, Store location, Webflow mirror (writable, read-only, unreadable), and
  Tested MCP version; the "Agent Instructions" table records the pointer.
- `references/rules.md` rule 19: the toolkit writes guidance to Agent
  Instructions and nothing else.
- Preflight records the version string `webflow_guide_tool` returns and says
  so once when it differs from the tested version.
- `manifest.py` accepts `cowork` as a surface.

### Added

- `references/stores.md`: data kinds, the one layout on every store, the
  adapter interface (`list`, `read`, `write`) with Notion, working folder,
  downloads, and the Webflow mirror, the size
  rule, the bundle format (schema 2, now carrying `runs` and `candidates`),
  `org.json`, the Webflow pointer block, the discovery order, and the failure
  and concurrency rules. The Notion mapping attaches the verbatim file
  because a rich text `text.content` is capped at 2000 characters and a block
  array at 100 elements (verified 2026-09-16 against the Notion API request
  limits).
- `references/org.json`: the organization configuration, shipped empty
  (`checks/repo_check.py` fails if the public plugin carries any
  organization's values).
- `flows/doctor.md` ("check access"): one read-only probe per capability and
  one store read, each classified by the access table, printed as a
  capability matrix with the exact ask for each refusal.
- **Cowork** as a surface (`SKILL.md` Surfaces table, README install), and the
  plugin's `userConfig` in `plugin.json` mirroring `org.json` so an
  administrator can preset the store in managed settings under
  `pluginConfigs`.
- README: "Set up for an organization (admin, once)", a Troubleshooting table
  in user wording, the new store table.
- Tests: `test_skill_tree.py` asserts the store specification, the empty
  `org.json`, the doctor probes, the write-ahead rule, that no flow reads or
  writes a record under the instruction prefix, and that the plugin's
  `userConfig` mirrors `org.json` with no sensitive field.

### Known limitations

- **Cowork is untested live.** Skill invocation, the plugin MCP OAuth, and
  Python availability in the Cowork VM are documented from the product
  documentation, not from a run; every script has a written fallback.
- **Confluence is not supported.** Notion is the only page store; a team on
  Confluence uses a working folder or downloads.
- **No `scripts/org_config.py`.** The organization zip is produced by the
  written procedure in `flows/onboard.md` step 9.

## [1.0.1] - 2026-09-16

### Fixed

- **Access diagnosis.** A 403 `forbidden` on `search_instructions` ("you cannot
  read this SiteAgentInstructions") is a Webflow **site-role** gate: Site
  managers and Designers read and manage Agent Instructions, Marketers and
  Content editors read them, Reviewers and Enterprise custom roles cannot read
  them. The skill used to blame two missing OAuth scopes and tell users to ask
  an admin to grant them, which no admin can do (Webflow's own OAuth app
  requests the scopes; a real scope error has the code `missing_scopes`). The
  wrong wording was in nine files and a test asserted it.
- `references/unsupported.md` gains the **access and entitlement table**: one
  row per signal (site role, resource permission `insufficient_permissions`,
  OAuth `missing_scopes`, plan `not_enterprise_plan_site`, `ModeForbidden`,
  429), each with its cause class, who fixes it, the exact ask, and what the
  skill does meanwhile. Every flow now points there instead of carrying its
  own remedy.
- `SKILL.md` preflight, `flows/onboard.md` (step 0 probe, step 8, step 11),
  `flows/build.md` (Phase 0 step 2, Phase 5 step 9), `flows/sync.md` step 0,
  `flows/resume.md`, `flows/maintain.md`, `references/rules.md` rule 19, and
  `references/webflow-conventions.md` say "site role" where they said "scope".
- `references/webflow-conventions.md`: "Token scopes observed" is now **Access
  observed** and records, per refused call, the status, the error `code`, the
  verbatim message, and the access-table row it matched, instead of a
  permission name to ask for. The "Agent Instructions" table gains the
  connector user's site role (asked at onboarding step 1; the MCP server has
  no whoami) and the probe's classification.
- The repository README's Requirements section explains the role model, and a
  new "Asking your Webflow admin for access" section carries a request
  template.
- Tests: `test_skill_tree.py` asserts the access table and that the OAuth
  scope names appear nowhere in the skill except the table's `missing_scopes`
  row; `checks/repo_check.py` enforces the same across the repository.

## [1.0.0] - Unreleased

First public release. The skill is site-agnostic: it ships the flows, the
rulebook, the schemas, and the scripts, and generates everything site-specific
at onboarding.

### Added

- `SKILL.md` entry point, explicit invocation only
  (`disable-model-invocation: true`).
- Flows: `onboard` (run first), `build` (Phases 0 to 7), `maintain`, `sync`,
  `resume`. Onboarding picks a store in its step 0 - Webflow Agent
  Instructions (the default on claude.ai and wherever there is no repository),
  a git repository (the reviewed path), or downloads only - measures the site
  identically in all three, and hands over a download bundle either way.
- The public-exposure guarantee: rulebook rule 19, a "What this can and cannot
  make public" section in `SKILL.md` and the repository README, a warning and a
  confirmation before every asset upload, an `alreadyPublic` list in
  `manifest.py ships` and in the build report, and a one-time check at
  onboarding that the instruction store does not appear in published output.
- References: the 19-rule Webflow MCP rulebook (rule 19 is the
  public-exposure guarantee), the interview question bank,
  the brief schema, the run manifest schema, the outline rendering spec, the
  catalog entry format, the candidates process, the unsupported-features list,
  and `webflow-conventions.md` as a template whose MEASURE rows are onboarding's
  worklist.
- Scripts (Python 3, standard library only): `validate_brief.py`,
  `render_outline.py`, `diff_inventory.py`, `catalog_lint.py`, `manifest.py`,
  `build_snapshot.py`, with `unittest` coverage and fixtures under
  `scripts/tests/`.
- A worked example for a fictional site: one catalog family and a filled
  conventions file in `references/examples/`, and a brief, a rendered outline,
  and a dry-run manifest in `assets/examples/`.
- `agents/openai.yaml` for Codex (`allow_implicit_invocation: false`, Webflow
  MCP dependency) and `.mcp.json` for Claude Code.
- `checks/repo_check.py` (via `checks/repo-check.sh`) at the repository root:
  manifest and metadata structure, plus a disclosure guard that fails on a
  Bridge App launch link, a secret-shaped token, or an identifier outside the
  synthetic set.
- Phase 6 normative checklist (outline, family rules, CTA, SEO, guard,
  tracking) recorded on the manifest; link extras match the measured site
  convention only; missing tracking is Designer handoff, never a silent create.

### Changed

- Plugin namespace is `temperage`. Invoke `/temperage:webflow-template`. The
  skill name is still `webflow-template`.

### Known limitations

- **The catalog ships empty.** Nothing works end to end until
  `flows/onboard.md` has run against a real site. That is deliberate; a catalog
  is not portable.
- **Branch-mode writes are unverified.** Onboarding can establish that element
  reads accept a branch page id. Whether writes on a branch page and
  `create_branch` without the Designer work is unknown until someone runs a
  branch-mode build and reports back.
- **The rate-limit behaviour in rule 10 is a shape, not a number.** The
  conservative defaults are a starting point; the real pacing for a site is
  whatever onboarding measures. A small site may need none of it.
- **Only the Webflow MCP server's own surface is covered.** Interactions,
  Google and Adobe fonts, roles and access, CMS schema changes, and publishing
  are manual Designer work (`references/unsupported.md`).
- **The worked example is fictional.** It exercises the scripts and shows the
  formats; it is not evidence that a build succeeds on any particular site.
