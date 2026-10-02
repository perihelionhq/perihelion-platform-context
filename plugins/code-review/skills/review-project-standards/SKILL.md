---
name: review-project-standards
description: Review a code change against the rules the project has written down in its standards and agent instruction files, including frontmatter requirements, reference and inclusion conventions, cross-references, platform portability, tool-use instructions, naming and placement, writing style, and protected paths. Use when reviewing for project standards compliance, especially changes to skills, rules, plugin manifests, or AGENTS.md and CLAUDE.md files.
---

# Review lens: Project Standards

Review the change against rules the project has explicitly written down, not invented rules or generic best practice. Every finding cites a specific rule from a specific file.

## Scope

The criteria are whatever the governing files state. The shapes below show how written rules about agent context (skills, rules, agent definitions, plugin manifests, `AGENTS.md`, `CLAUDE.md`) typically get violated:

- **Frontmatter** Missing required fields such as `name` or `description`, descriptions that do not follow the stated format, names that do not match their directory.
- **Reference inclusion** Links, backtick paths, or `@` inline includes used where the standards require a different mode for that kind of file.
- **Cross-references** Agent or skill names that are not qualified as required, slash-command syntax where the standards call for semantic wording, tools named by platform-specific names without the capability class.
- **Portability** Platform-specific tool names without equivalents, or assumptions about tool availability that break on other agents.
- **Tool-use instructions** Shell commands instructed for routine file discovery, search, or reading where the standards require native tools; chained commands or error suppression where the standards ask for one simple command at a time.
- **Naming and structure** Files in the wrong directory, names that break the stated convention, README tables or counts not updated when components are added or removed.
- **Writing style** Second person where the standards require imperative form; hedge words such as `might`, `could`, or `consider` where the standards require clear directives.
- **Protected artifacts** Instructions that delete or gitignore paths the standards designate as protected.

## Method

Judge each changed file only against the criteria files that govern it. Find every `AGENTS.md`, `CLAUDE.md`, and designated standards file (such as `CODING_STANDARDS.md`) whose directory is an ancestor of the changed file: a root file governs the whole repository, a nested one only its subtree. Where a designated standards file governs a path, it is the criteria source; instruction files supply criteria only for paths no standards file covers.

Extract the rules whatever shape the file takes, prose, bullets, tables, or headings. Apply only the sections that address the changed file types: a skill checklist does not govern a TypeScript change, and a commit convention does not govern markdown content.

## Threshold

Report a violation when you can quote the rule and point to the line in the change that breaks it. Report a judgment-dependent application, such as whether a description adequately says when to use a skill, only when the rule is clear and the gap is unambiguous.

Do not report rules that do not apply to the changed file type, violations automated checks already catch, pre-existing violations on lines the change did not touch, generic best practices no governing file states, how the criteria files themselves are formatted, or opinions on the quality of the rules.

## Reporting

- Quote the rule, or cite its section, and name the file it comes from.
- Cite the specific line or lines in the change that violate it.
- State what would bring the line into compliance. A finding without both a cited rule and a cited violation is not a finding.
