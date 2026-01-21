# Changelog - Template Migration

## 2026-01-21: Template-Based Workflow Migration

### Added
- Template extraction functionality using `adm-zip`
- Random slug generation for unique deck folders
- HTML cleaning and customization functions
- SLConfig metadata updating
- `--slug` command-line option
- Template structure documentation
- Chart.js setup instructions for template

### Changed
- `create-presentation.js`: Complete rewrite - now extracts template instead of generating HTML
- `--output` → `--output-dir`: Now creates folder instead of single file
- CSS customization workflow: Edit `lib/offline-v2.css` instead of copying `base-styles.css`
- Output structure: `deck-{slug}/` folders instead of single HTML files
- Dependencies: Added `adm-zip` and `puppeteer` to package.json

### Updated Documentation
- `SKILL.md`: Updated all workflow steps for template-based approach
- `advanced-features.md`: Added note about template plugins
- `charts.md`: Added Chart.js setup instructions and warnings

### Removed
- CDN-based HTML generation
- `base-styles.css` copying functionality
- Chart.js auto-inclusion (now manual)

### Migration Notes
- Backup files saved in `agent-docs/template-migration-2026-01-21/`
- All existing functionality preserved, just different implementation
- Check scripts (`check-overflow.js`, `check-charts.js`) work unchanged with new paths
