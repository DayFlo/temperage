# Changelog

Notable changes to this repository: the marketplace root, the
`temperage` plugin, and the checks. The `webflow-template` skill keeps
its own changelog at
`plugins/temperage/skills/webflow-template/CHANGELOG.md`, and that is
the version `SKILL.md` mirrors.

The format follows Keep a Changelog; versions follow semver.

## [Unreleased]

### Changed

- **Marketplace renamed to `temperage`**, matching the repository
  (`DayFlo/temperage`) and the plugin. Both `.claude-plugin/marketplace.json`
  and `.agents/plugins/marketplace.json` now carry `temperage`; the install
  id is `temperage@temperage`. The READMEs, `plugin.json` homepage, and
  `checks/repo_check.py` follow. Anyone who added the marketplace under its
  old name removes it and adds `DayFlo/temperage` again.

## [1.1.1] - 2026-09-22

### Fixed

- **Plugin configuration reaches the skill.** Cowork left every
  `${user_config.*}` token in the flows and references literal: substitution
  happens in `SKILL.md` only. `SKILL.md` gains an "Administrator
  configuration" table with one token per `userConfig` field, and the flows,
  `references/stores.md`, and onboarding point there instead of carrying
  tokens of their own. A test fails if a flow carries a token.
- **Designer launch on Cowork.** The session's `open` runs in a cloud
  container and cannot reach the Mac. `SKILL.md`, `flows/build.md` Phase 0
  step 5, and the conventions template now say to use a Mac-side browser tool
  (for example Control Chrome's `open_url`), otherwise the Claude.ai link.

## [1.1.0] - 2026-09-16

### Changed

- The `webflow-template` skill's storage model: guidance is mirrored into
  Webflow Agent Instructions, records never are, and the source of truth is
  Notion, a working folder, or downloads. Details in the skill
  changelog.
- `plugin.json` declares `userConfig` (source of truth, location, instruction
  prefix, allowed stores, sites, tested MCP version), all non-sensitive, so an
  administrator can preset the store in managed settings under
  `pluginConfigs`.
- README: Cowork install, "Set up for an organization (admin, once)", the
  store table, Troubleshooting.

### Added

- `checks/repo_check.py`: `references/org.json` ships empty; `stores.md` and
  `flows/doctor.md` exist and `SKILL.md` names them; `userConfig` mirrors
  `org.json` with no sensitive field.

## [1.0.1] - 2026-09-16

### Fixed

- The skill's diagnosis of a 403 on Agent Instructions: it is a Webflow
  site-role gate, not a missing OAuth scope. Details in the skill changelog.
- README: Requirements explains which built-in site roles can read and write
  Agent Instructions; new "Asking your Webflow admin for access" template.
- `checks/repo_check.py`: new check that `references/unsupported.md` carries
  the access and entitlement table and that the OAuth scope names appear
  nowhere else in the repository.

## [1.0.0] - Unreleased

First public release.

### Added

- Marketplace root: `.claude-plugin/marketplace.json` for Claude Code and
  `.agents/plugins/marketplace.json` for Codex, both named
  `webflow-template-skill`.
- The `temperage` plugin at `plugins/temperage`, carrying the
  `webflow-template` skill and a `.mcp.json` for the Webflow MCP server.
  Invoke `/temperage:webflow-template`.
- `checks/repo_check.py`, run by the `checks/repo-check.sh` wrapper: manifests
  parse and carry the keys Claude Code and Codex need; the documented command
  form is the namespaced one; the skill version matches its changelog; the
  shipped catalog is empty; the licence names a copyright holder. Then the
  disclosure guard: no Bridge App launch link, no long secret-shaped token, no
  home-directory path, no branch staging domain, and no Webflow-shaped
  identifier outside the repository's synthetic set. It uses no git and no
  network so it runs in any checkout.
- `.github/workflows/ci.yml`: unit tests, catalog lint on the shipped
  (empty) catalog and on the worked example, brief validation, and
  `checks/repo-check.sh`.
- MIT `LICENSE`, Copyright (c) 2026 Dayton Floyd.
- A public-exposure guarantee that cannot be quietly deleted: `repo_check.py`
  fails unless the "What this can and cannot make public" section is present in
  both `README.md` and the skill's `SKILL.md`, the rulebook still forbids
  `publish_site`, and the uploaded-asset warning is still in the rulebook and
  the build flow.
- README install for Claude.ai: zip the `webflow-template` skill folder (folder
  name must match the skill name), upload at Customize → Skills, or have a
  Team/Enterprise owner provision it under Organization settings → Skills.
  Claude.ai does not clone this repository.
- Phase 6 normative checklist and site-matched tracking: verify by readback
  table (PASS/WARN/FAIL/HANDOFF); copy only link extras the site already uses;
  tracking gaps hand off to the Designer.

### Changed

- Plugin renamed from `design-automations` to `temperage`. Folder is
  `plugins/temperage`. Invoke is `/temperage:webflow-template`.
- README: short pitch then install, usage, onboarding, the public-exposure
  table, and a safety list that does not repeat the table. Dropped the
  verified/not-verified essay (one line under Development). Plugin folder
  now has its own `plugins/temperage/README.md`.

### Notes

The skill is site-agnostic by construction. Nothing about any particular
Webflow site is baked into the flows, rulebook, schemas, or scripts: every
fact a build depends on is measured at onboarding and recorded in
`references/webflow-conventions.md`. No catalog, no site inventory, and no run
records are shipped, and every identifier in the tree is synthetic.
