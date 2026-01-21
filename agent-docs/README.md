# Agent Documentation

This directory contains documentation and backups from agent-assisted development work.

## Structure

- `template-migration-2026-01-21/` - Documentation and backups from the template-based workflow migration
  - `MIGRATION_SUMMARY.md` - Comprehensive summary of the migration
  - `CHANGELOG.md` - Changelog of what was added/changed/removed
  - `create-presentation.js.bak` - Backup of original script (CDN-based)
  - `SKILL.md.bak` - Backup of original documentation (CDN-based workflow)

- `examples/` - Example presentations and prompts
  - `prompt.md` - Example prompt for creating presentations
  - `revealjs/` - Example reveal.js presentation
  - `pptx/` - Example PowerPoint-style presentation

- `demo_example/` - Demo examples

## Recent Work

### Template Migration (2026-01-21)

Migrated the reveal.js skill from generating HTML from scratch to extracting and customizing templates from `template.zip`. See `template-migration-2026-01-21/MIGRATION_SUMMARY.md` for full details.

**Key Changes:**
- Script now extracts template.zip instead of generating HTML
- Output changed from single HTML file to `deck-{slug}/` folders
- All dependencies included locally (no CDN)
- Chart.js no longer included by default

**Backup Files:**
- Original `create-presentation.js` (CDN-based generation)
- Original `SKILL.md` (CDN-based workflow documentation)
