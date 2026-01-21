# Template-Based Workflow Migration

**Date:** January 21, 2026  
**Task:** Rewrite reveal.js skill to use template extraction instead of HTML generation

## Overview

Migrated the reveal.js skill from generating HTML presentations from scratch (using CDN links) to extracting and customizing a clean template from `template.zip`. This aligns the skill with slides.com export format and ensures better compatibility.

## Key Changes

### 1. Script Rewrite: `create-presentation.js`

**Before:** Generated HTML from scratch with CDN links for reveal.js, Chart.js, Font Awesome, etc.

**After:** Extracts `template.zip` to `deck-{slug}/` folders and customizes the template HTML.

**New Features:**
- Extracts template.zip to unique `deck-{slug}/` folders (random 6-char hex slug)
- Cleans template HTML, removing example content
- Generates new slide structure based on `--structure` parameter
- Updates SLConfig metadata in template
- Changed `--output` to `--output-dir` (now creates a folder, not a single file)
- Added `--slug` option for custom folder names

**Key Functions Added:**
- `generateSlug()` - Creates random 6-character hex string
- `extractTemplate(slug, outputDir)` - Extracts zip and organizes files
- `cleanTemplateHTML(html, options)` - Removes template content, adds new slides, updates config

### 2. Dependencies Updated: `package.json`

**Added:**
- `adm-zip: ^0.5.16` - For extracting template.zip
- `puppeteer: ^21.0.0` - For overflow checking (was missing)

**Existing (no changes):**
- `cheerio: ^1.1.2` - For HTML parsing
- `decktape` - For screenshots
- `playwright: ^1.57.0` - For testing

### 3. Documentation Updates: `SKILL.md`

**Step 2 - Changed from "Generate Scaffold" to "Extract Template":**
- Updated command examples to show `--output-dir` instead of `--output`
- Added explanation of what gets created (`deck-{slug}/` folder structure)
- Documented `--slug` option
- Explained that template includes all dependencies (no CDN needed)

**Step 3 - Updated CSS Customization:**
- Removed references to `base-styles.css` (template has its own CSS)
- Explained template includes local fonts in `lib/fonts/`
- Updated instructions for customizing `lib/offline-v2.css`
- Added example for adding custom CSS file

**Step 4 - Emphasized HTML Markup:**
- Added note: "Always use HTML markup, never markdown" (even though template includes markdown plugin)

**Step 5 - Updated Paths:**
- Changed examples from `presentation.html` to `deck-{slug}/index.html`

**Step 6 - Updated Screenshot Paths:**
- Changed from `cd <presentation-directory>` to `cd deck-{slug}`

**New Section Added - "Template Structure":**
- Documents what's in template.zip
- Lists available plugins (RevealZoom, RevealNotes, RevealMarkdown, RevealHighlight)
- Notes that template uses slides.com format but converts to standard reveal.js sections

**Dependencies Section Updated:**
- Added adm-zip and cheerio to required dependencies list

**Charts Section Updated:**
- Note that Chart.js is NOT included in template by default

### 4. Reference Documentation Updates

#### `advanced-features.md`
- Added note at top: "The template includes RevealZoom, RevealNotes, RevealMarkdown, and RevealHighlight plugins. All features documented here work with the template."

#### `charts.md`
- Added warning at top: "Chart.js is NOT included in the template by default"
- Added new section "Adding Chart.js to Template" with step-by-step instructions:
  1. Add Chart.js script tags
  2. Add RevealChart to plugins array
  3. Configure chart defaults
  4. Optional: Disable animations for PDF export
- Updated "Setup" section to reference the new instructions

### 5. Check Scripts

**`check-overflow.js`** and **`check-charts.js`** - No code changes needed. They work with any HTML file path, just need to be called with the correct path: `deck-{slug}/index.html`

## Template Structure

The `template.zip` file contains:
- `index.html` - Presentation HTML with slides.com format
- `lib/reveal.js` - reveal.js core library
- `lib/reveal.css` - reveal.js styles
- `lib/reveal-plugins.js` - Plugins bundle
- `lib/offline-v2.css` - Complete CSS including themes
- `lib/offline.js` - Offline support
- `lib/fonts/` - Local font files (Montserrat, Overpass, Lato, Open Sans, etc.)

## Migration Benefits

1. **Offline Support** - All dependencies included, no CDN required
2. **slides.com Compatibility** - Uses same format as slides.com exports
3. **Consistent Structure** - Every presentation has same folder structure
4. **Plugin Availability** - Zoom, Notes, Markdown, Highlight plugins pre-configured
5. **Local Fonts** - No need for Google Fonts CDN
6. **Unique Folders** - Each presentation gets its own `deck-{slug}/` folder

## Breaking Changes

1. **Command Line Interface:**
   - `--output <file>` → `--output-dir <dir>` (now creates folder, not file)
   - New `--slug` option for custom folder names

2. **Output Structure:**
   - Before: Single HTML file + optional CSS file
   - After: `deck-{slug}/` folder with `index.html` and `lib/` subfolder

3. **CSS Customization:**
   - Before: `base-styles.css` copied to presentation directory
   - After: Edit `lib/offline-v2.css` or add custom CSS file

4. **Chart.js:**
   - Before: Included by default in generated HTML
   - After: NOT included in template, must be added manually

## Files Modified

1. `skills/revealjs/scripts/create-presentation.js` - Complete rewrite
2. `skills/revealjs/SKILL.md` - Updated workflow sections
3. `skills/revealjs/references/advanced-features.md` - Added template plugin note
4. `skills/revealjs/references/charts.md` - Added Chart.js setup instructions
5. `package.json` - Added adm-zip and puppeteer dependencies

## Backup Files

Original versions saved in this directory:
- `create-presentation.js.bak` - Original script (generated HTML from scratch)
- `SKILL.md.bak` - Original documentation (CDN-based workflow)

## Testing Recommendations

1. Test template extraction with various slug options
2. Verify HTML cleaning removes template content correctly
3. Test SLConfig updates work properly
4. Verify all plugins from template work
5. Test check-overflow.js and check-charts.js with new paths
6. Verify Chart.js can be added manually when needed

## Future Considerations

1. Consider adding Chart.js to template if charts become commonly used
2. May want to add option to include/exclude specific plugins
3. Could add template validation to ensure template.zip structure is correct
4. Consider adding option to use CDN instead of local files for smaller file sizes
