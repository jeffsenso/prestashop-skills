# PrestaShop Module Development Tools - Skills

This directory contains AI skills for PrestaShop module development.

## Available Skills

### prestashop-module-development
Complete workflow for creating modern PrestaShop modules and updating legacy code to modern patterns.

**Features:**
- Modern Symfony-based architecture
- **Mandatory installer pattern** with explicit install/uninstall ordering, hooks and tabs as class property arrays, parent group tab auto-installation
- Security best practices
- Legacy code conversion patterns
- Database operations with Doctrine
- Translation and multilingual support, including `TranslatableType` per-language fields
- Theme template injection for PS8 (marker-based file patching)
- Pre-built JS bundle templates for grid and translatable forms
- Performance optimization
- Testing and validation
- Steering system: load-order rules for layering project-specific context files on top of the base skill

**Usage:**
This skill can be used by AI assistants like Claude, ChatGPT, and others to help developers create high-quality PrestaShop modules following current best practices.

The skill is company-neutral and self-contained for core features — all examples use generic placeholders, and essential documentation (like CategoryChoiceTreeType setup) is embedded directly in the skill. Company or project-specific conventions belong in a separate steering layer loaded alongside the skill (see `## Steering` in `SKILL.md`).

**Documentation:**
Based on official PrestaShop documentation:
- https://devdocs.prestashop-project.org/8/modules/creation/
- https://devdocs.prestashop-project.org/9/modules/creation/

**Submission Ready:**
Structured for submission to https://skills.sh/ following established patterns.