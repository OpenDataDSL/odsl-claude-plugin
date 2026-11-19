# OpenDataDSL plugins for Claude

This repository is a Claude plugin marketplace. It holds one plugin, `opendatadsl`, which gives Claude:

- **The `odsl-scripting` skill**: how to write, fix and review OpenDataDSL (ODSL) scripts, with the language, function, type, service, calendar and automation references.
- **The `odsl-insights-extensions` skill**: how to build insight reports and portal extensions in the house style, with a tested example page, the data API, the manifest, policies and publishing.
- **The Fusion AI connector**: Claude can find your data, curves, scripts and processes, validate scripts, and (when you ask) save them, all under your own OpenDataDSL sign-in.

## Install

**Claude Team or Enterprise (whole organisation).** An organisation owner adds this repository (https://github.com/OpenDataDSL/odsl-claude-plugin) as a plugin marketplace in Organization settings, then chooses whether the `opendatadsl` plugin is installed by default or available for people to install.

**Claude Code.**

```
/plugin marketplace add OpenDataDSL/odsl-claude-plugin
/plugin install opendatadsl@opendatadsl
```

**Skills only.** Zip a skill folder (for example `plugins/opendatadsl/skills/odsl-scripting`) and upload it in Claude under Customize > Skills.

The first time Claude needs your data, the Fusion AI connector asks you to sign in with your normal Microsoft/Entra account using a code and a link. See https://doc.opendatadsl.com/docs/product/mcp/signing-in.

## Layout

```
.claude-plugin/marketplace.json            marketplace listing
plugins/opendatadsl/
  .claude-plugin/plugin.json               plugin manifest, including the Fusion AI MCP server
  .claude-plugin/icon.png                  listing icon
  skills/odsl-scripting/
    SKILL.md                               workflow and language essentials
    references/practical-notes.md          hand-written: behaviours that break scripts
    references/*.md                        generated from the public documentation
  skills/odsl-insights-extensions/
    SKILL.md                               what to build, workflow, key rules, naming
    references/house-style.md              hand-written: page structure, data API, Mustache rules
    references/example-insight.md         hand-written, tested example page
    references/insight-report-style.md    copy of the #insight-report-style partial
    references/*.md                        generated from the public documentation
```

## Updating the references

The generated reference files come from the documentation repository, which also holds the generator, `tools/build-claude-skill-refs.py` (see the comment at its top). Run it after a documentation release.

The hand-written files (`practical-notes.md`, `house-style.md`, `example-insight.md`) are maintained by hand. `insight-report-style.md` is a copy of the platform's `#insight-report-style` partial: copy it again when the partial changes. Bump `version` in `plugin.json` and `marketplace.json` when you publish a change.
