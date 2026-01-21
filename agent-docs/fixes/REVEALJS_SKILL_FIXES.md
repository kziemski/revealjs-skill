# Suggestions for Fixing Reveal.js Agent Skill

## Issues Encountered

### 1. CSS File Management

**Problem**: Custom CSS was created as a separate file (`mufg-custom.css`) then required manual integration.

**Root Cause**: Skill workflow doesn't have a CSS integration phase. It creates separate custom CSS but doesn't append it to `lib/offline-v2.css`.

**Suggested Fix**:
- Add CSS integration step before HTML content creation
- Option 1: Append custom CSS directly to `lib/offline-v2.css` during template customization
- Option 2: Create custom CSS, then auto-merge it into main CSS file
- Option 3: Provide parameter flag `--integrate-css` to control behavior

```javascript
// In create-presentation.js or new integrate-css.js script
async function integrateCustomCSS(deckPath) {
  const customCss = fs.readFileSync(path.join(deckPath, 'mufg-custom.css'), 'utf8');
  const mainCss = fs.readFileSync(path.join(deckPath, 'lib/offline-v2.css'), 'utf8');
  fs.writeFileSync(
    path.join(deckPath, 'lib/offline-v2.css'),
    mainCss + '\n' + customCss
  );
  fs.unlinkSync(path.join(deckPath, 'mufg-custom.css'));
}
```

### 2. HTML CSS References

**Problem**: HTML file contained reference to `mufg-custom.css` which needs manual removal.

**Root Cause**: Template HTML generation doesn't account for custom CSS file deletion.

**Suggested Fix**:
- Modify `index.html` to only reference `lib/offline-v2.css` from the start
- OR add post-processing step to remove any `*.css` file references except `lib/offline-v2.css`

```javascript
// In create-presentation.js or clean-html.js
function removeExternalCssReferences(htmlPath) {
  let html = fs.readFileSync(htmlPath, 'utf8');
  html = html.replace(/<link[^>]*href="[^"]*\.css"[^>]*>/g, (match) => {
    if (match.includes('lib/offline-v2.css')) return match;
    return '';
  });
  fs.writeFileSync(htmlPath, html);
}
```

### 3. Missing Assets Folder

**Problem**: Presentation structure didn't include an `assets/` folder, which is a common requirement for presentations.

**Root Cause**: Skill workflow doesn't create standard directory structure (assets/, images/, etc.).

**Suggested Fix**:
- Add `assets/` folder creation as part of template extraction
- Provide option `--create-assets` to include it

```javascript
// In create-presentation.js
function createStandardDirectories(deckPath) {
  const dirs = ['assets', 'images', 'docs'];
  dirs.forEach(dir => {
    const dirPath = path.join(deckPath, dir);
    if (!fs.existsSync(dirPath)) {
      fs.mkdirSync(dirPath, { recursive: true });
    }
  });
}
```

### 4. Build Artifacts Left Behind

**Problem**: `screenshots/` directory and `output.pdf` file remain after creation.

**Root Cause**: Skill doesn't have cleanup phase to remove temporary build artifacts.

**Suggested Fix**:
- Add `--clean` flag or automatic cleanup after screenshot verification
- Create separate utility script for cleanup

```javascript
// In cleanup.js or integrated into skill
function cleanBuildArtifacts(deckPath) {
  const artifacts = ['screenshots', 'output.pdf'];
  artifacts.forEach(artifact => {
    const artifactPath = path.join(deckPath, artifact);
    if (fs.existsSync(artifactPath)) {
      if (fs.lstatSync(artifactPath).isDirectory()) {
        fs.rmSync(artifactPath, { recursive: true, force: true });
      } else {
        fs.unlinkSync(artifactPath);
      }
      console.log(`Removed: ${artifact}`);
    }
  });
}
```

### 5. Plan Mode Confusion

**Problem**: Agent entered plan mode unexpectedly after user provided clear requirements, causing delay.

**Root Cause**: System automatically activated plan mode without explicit user request for planning phase.

**Suggested Fix**:
- Only activate plan mode when user explicitly requests it (e.g., "create a plan for...")
- OR make plan mode opt-in with clear messaging
- Add detection: if user gives specific file paths and clear actions, skip plan mode

### 6. Workflow Gaps

**Problem**: Multiple manual steps required (CSS integration, HTML cleanup, artifact removal).

**Root Cause**: Skill lacks end-to-end workflow automation.

**Suggested Fix**: Create unified workflow scripts:

```bash
# Add to package.json scripts
{
  "scripts": {
    "create": "node scripts/create-presentation.js",
    "integrate-css": "node scripts/integrate-css.js",
    "clean": "node scripts/clean-artifacts.js",
    "finalize": "npm run integrate-css && npm run clean"
  }
}

# Or single comprehensive script
node scripts/build-final.js --title "My Deck" --integrate-css --clean --create-assets
```

### 7. CSS File Read for HTML Edit

**Problem**: Attempted to edit HTML without reading it first, causing error.

**Root Cause**: Agent tool restriction requires reading file before editing.

**Suggested Fix**:
- This is correct behavior for safety, but agent should anticipate this
- Add better error handling or pre-read workflow

```javascript
// Better pattern in agent logic
function safeEdit(filePath, oldString, newString) {
  const content = fs.readFileSync(filePath, 'utf8');
  if (!content.includes(oldString)) {
    console.warn(`Pattern not found in ${filePath}`);
    return false;
  }
  const newContent = content.replace(oldString, newString);
  fs.writeFileSync(filePath, newContent);
  return true;
}
```

## Recommended Workflow Improvements

### Enhanced Template Creation Command

```bash
# Proposed new command structure
node scripts/create-presentation.js \
  --title "Strategic Architecture" \
  --structure 1,d,3,1,3,1,3,1,2,1 \
  --brand mufg \
  --integrate-css \
  --create-assets \
  --output-dir ./presentations \
  --clean-artifacts
```

### New Script: `finalize.js`

```javascript
#!/usr/bin/env node

/**
 * Finalize presentation - Integrate styles and clean artifacts
 */

const fs = require('fs');
const path = require('path');

const deckPath = process.argv[2] || '.';

console.log('Finalizing presentation...');

// 1. Ensure assets folder exists
const assetsPath = path.join(deckPath, 'assets');
if (!fs.existsSync(assetsPath)) {
  fs.mkdirSync(assetsPath, { recursive: true });
  console.log('✓ Created assets folder');
}

// 2. Integrate custom CSS if exists
const customCssPath = path.join(deckPath, 'mufg-custom.css');
if (fs.existsSync(customCssPath)) {
  const customCss = fs.readFileSync(customCssPath, 'utf8');
  const mainCssPath = path.join(deckPath, 'lib/offline-v2.css');
  const mainCss = fs.readFileSync(mainCssPath, 'utf8');
  fs.writeFileSync(mainCssPath, mainCss + '\n' + customCss);
  fs.unlinkSync(customCssPath);
  console.log('✓ Integrated and removed custom CSS');
}

// 3. Remove CSS reference from HTML
const htmlPath = path.join(deckPath, 'index.html');
let html = fs.readFileSync(htmlPath, 'utf8');
html = html.replace(/<link[^>]*href="[^"]*\.css"[^>]*>/g, (match) => {
  if (match.includes('lib/offline-v2.css')) return match;
  return '';
});
fs.writeFileSync(htmlPath, html);
console.log('✓ Updated HTML CSS references');

// 4. Remove build artifacts
const artifacts = ['screenshots', 'output.pdf'];
artifacts.forEach(artifact => {
  const artifactPath = path.join(deckPath, artifact);
  if (fs.existsSync(artifactPath)) {
    if (fs.lstatSync(artifactPath).isDirectory()) {
      fs.rmSync(artifactPath, { recursive: true, force: true });
    } else {
      fs.unlinkSync(artifactPath);
    }
    console.log(`✓ Removed ${artifact}`);
  }
});

console.log('\n✓ Presentation finalized successfully!');
```

### Updated SKILL.md Documentation

Add section to SKILL.md:

```markdown
## Finalization Workflow

After creating your presentation content, run the finalization script:

```bash
node scripts/finalize.js /path/to/deck-folder
```

This script will:
1. Create an `assets/` folder if it doesn't exist
2. Integrate any custom CSS (`*.css` files) into `lib/offline-v2.css`
3. Remove external CSS references from `index.html`
4. Clean up build artifacts (`screenshots/`, `output.pdf`)

### Manual Finalization Steps

If you prefer to finalize manually:

1. **Integrate CSS**: Append custom CSS to main file
2. **Update HTML**: Remove external CSS references
3. **Create Assets**: Make `assets/` folder for images/docs
4. **Clean Up**: Remove `screenshots/` and `output.pdf`
```

## Summary of Priority Fixes

### High Priority
1. ✅ CSS integration automation
2. ✅ HTML reference cleanup
3. ✅ Assets folder creation
4. ✅ Artifact cleanup script

### Medium Priority
5. ⚠️ Plan mode detection improvement
6. ⚠️ Better error handling for file edits

### Low Priority
7. 💡 Enhanced command-line options
8. 💡 Comprehensive documentation updates

## Testing Recommendations

Before considering skill fixed, test:

```bash
# 1. Create presentation
node scripts/create-presentation.js --structure 1,d,2,1 --title "Test"

# 2. Add custom CSS
cat > deck-xyz/custom.css << 'EOF'
:root { --test: #fff; }
EOF

# 3. Run finalization
node scripts/finalize.js deck-xyz

# 4. Verify structure
ls -la deck-xyz/
# Should show: assets/ (empty), index.html, lib/, NO custom.css, NO screenshots/

# 5. Verify HTML
grep -c '<link.*css' deck-xyz/index.html
# Should return 1 (only lib/offline-v2.css)

# 6. Verify CSS
tail -20 deck-xyz/lib/offline-v2.css
# Should contain custom CSS variables

# 7. Test in browser
# Open index.html and verify styling works
```

## Conclusion

The reveal.js skill works well for initial presentation creation but lacks finalization automation. The suggested fixes address CSS integration, artifact cleanup, and standard directory structure to provide a complete, production-ready presentation workflow.