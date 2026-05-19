# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-05-19

### Added

- `SKILL.md` — new `## Steering` section: describes the steering layers system, load order, and conflict resolution rules so consumers can attach project-specific context files.

### Changed

- `SKILL.md` — company-specific rules (table prefix, admin tab group, namespace examples) replaced with generic placeholders; the skill is now company-neutral.
- `SKILL.md` — removed live URL from source attribution; replaced with a static note.
- `references/module-class-and-installer.md` — admin tab group and label examples made generic; added `// Replace with your own values` comments.
- `references/entity-doctrine.md` — table prefix, class name, and namespace examples made generic.
- `references/grid-system.md` — table name examples made generic.
- `references/module-structure.md` — namespace examples made generic.

### Security

- `SKILL.md` — removed live URL pointing to an external repository (static agent context URL, security scanner fix).
- `ps9-core-ai/Domain/Webservice/CONTEXT.md` — added static documentation-only disclaimer at top of file (security scanner fix).

## [1.1.0] - 2026-04-15

### Added

- `references/` folder with 15 domain-specific reference files (module structure, installer, configuration page, entities, grid, security, services split, translations, validation, debugging, and more).

### Changed

- `SKILL.md` — refactored from a monolithic file into a concise index; each step now delegates to a dedicated reference file. Added Grid system, services-split, and guard-pattern rules.

## [1.0.0] - 2026-04-02

### Added

- Initial release of the `prestashop-module-development` skill.
