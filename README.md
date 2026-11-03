# PrestaShop Module Development Tools - Skills

This directory contains AI skills for PrestaShop module development.

## Available Skills

### prestashop-module-development
Complete workflow for creating modern PrestaShop 8.x/9.x modules and updating legacy code to modern patterns.

**Features:**
- Modern Symfony-based architecture, services and dependency injection
- Mandatory installer pattern and YAML-based configuration management
- Grid system (CRUD lists, filters, toggle, drag-and-drop) with a generic Grid JS init script
- Forms, translations and multilingual support (`TranslatableType`)
- Cart rules, console commands, theme template injection
- Security, legacy code conversion, validation (lotr) and debugging
- PrestaShop 9 core documentation and steering system for project-specific context

Details: [skill README](skills/prestashop-module-development/README.md) — history: [CHANGELOG.md](CHANGELOG.md) (summary) and [skill CHANGELOG](skills/prestashop-module-development/CHANGELOG.md) (full).

**Usage:**
This skill can be used by AI assistants like Claude, ChatGPT, and others to help developers create high-quality PrestaShop modules following current best practices.

The skill is company-neutral and self-contained for core features. Company or project-specific conventions belong in a separate steering layer loaded alongside the skill (see `## Steering` in `SKILL.md`).

**Documentation:**
Based on official PrestaShop documentation:
- https://devdocs.prestashop-project.org/8/modules/creation/
- https://devdocs.prestashop-project.org/9/modules/creation/

**Submission Ready:**
Structured for submission to https://skills.sh/ following established patterns.
