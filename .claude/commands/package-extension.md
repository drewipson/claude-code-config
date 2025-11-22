# Package Extension

Build and package the VS Code extension as a .vsix file for distribution.

## Instructions

1. **Pre-packaging checklist:**
   - [ ] All changes committed
   - [ ] Version bumped in `package.json` (if releasing)
   - [ ] CHANGELOG.md updated with changes
   - [ ] README.md up to date
   - [ ] No debugging code or console.log statements
   - [ ] TypeScript compilation succeeds
   - [ ] Extension tested in Development Host

2. **Run production build:**
```bash
npm run build
```

This will:
- Compile TypeScript with strict mode
- Bundle with esbuild (minified)
- Output to `dist/extension.js`
- Generate source maps for debugging

3. **Verify build output:**
```bash
ls -lh dist/extension.js
```
- Check file size (should be optimized)
- Ensure file exists and is recent

4. **Package the extension:**
```bash
npx vsce package
```

This will:
- Validate package.json
- Validate all required files are included
- Create a `.vsix` file (e.g., `claude-code-manager-0.0.1.vsix`)

5. **Verify package contents:**
```bash
# Extract and inspect (optional)
unzip -l claude-code-manager-*.vsix

# Check package size
ls -lh claude-code-manager-*.vsix
```

## Version Management

Before packaging a release, bump the version in `package.json`:

```bash
# Patch release (0.0.1 -> 0.0.2) - Bug fixes
npm version patch

# Minor release (0.0.1 -> 0.1.0) - New features, backwards compatible
npm version minor

# Major release (0.0.1 -> 1.0.0) - Breaking changes
npm version major
```

This will:
- Update version in package.json
- Create a git commit
- Create a git tag

## Package.json Required Fields

Ensure these fields are properly set:

```json
{
  "name": "claude-code-manager",
  "displayName": "Claude Code Config",
  "description": "Unified management interface for Claude Code configurations",
  "version": "0.0.1",
  "publisher": "your-publisher-id",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/claude-code-manager"
  },
  "engines": {
    "vscode": "^1.85.0"
  },
  "categories": ["Other"],
  "keywords": ["claude", "claude-code", "configuration", "management"],
  "icon": "resources/icons/claude-logo.png",
  "license": "MIT"
}
```

## Testing the Package

Install the package locally to test:

```bash
# Install in VS Code
code --install-extension claude-code-manager-0.0.1.vsix

# Test the extension
# 1. Reload VS Code
# 2. Verify extension appears in Extensions view
# 3. Test all functionality
# 4. Check for errors in Developer Tools console

# Uninstall after testing
code --uninstall-extension your-publisher-id.claude-code-manager
```

## Publishing (Future)

When ready to publish to VS Code Marketplace:

```bash
# Login (one-time setup)
npx vsce login your-publisher-id

# Publish (automatically increments version and packages)
npx vsce publish

# Or publish specific version
npx vsce publish patch  # 0.0.1 -> 0.0.2
npx vsce publish minor  # 0.0.1 -> 0.1.0
npx vsce publish major  # 0.0.1 -> 1.0.0
```

## Files Included in Package

The `.vsixmanifest` includes:
- `dist/` - Compiled extension code
- `resources/` - Icons, templates, images
- `README.md` - Extension documentation
- `CHANGELOG.md` - Version history
- `LICENSE` - License file
- `package.json` - Extension manifest

Excluded (via `.vscodeignore`):
- `src/` - Source TypeScript files
- `node_modules/` - Dependencies (bundled in dist/)
- `.gitignore`, `.eslintrc`, etc. - Development config
- `tsconfig.json` - TypeScript config
- Test files

## Troubleshooting

**Validation errors:**
- Fix all issues reported by vsce
- Ensure all required package.json fields present
- Verify icon file exists and is correct format

**Package too large:**
- Check .vscodeignore excludes source files
- Ensure node_modules not included (should be bundled)
- Optimize icon/image sizes

**Extension doesn't activate:**
- Test in Extension Development Host first
- Check activation events in package.json
- Verify main entry point is correct: `dist/extension.js`

## Output

Show:
```
✓ Production build complete
  dist/extension.js (125 KB, minified)

✓ Package created
  claude-code-manager-0.0.1.vsix (142 KB)

Next steps:
1. Test installation: code --install-extension claude-code-manager-0.0.1.vsix
2. Verify all functionality works
3. Commit and tag: git commit -am "chore: release v0.0.1" && git tag v0.0.1
4. Push: git push && git push --tags
5. Create GitHub release and attach .vsix file
6. (Optional) Publish to marketplace: npx vsce publish
```

## Checklist

- [ ] Version bumped in package.json
- [ ] CHANGELOG.md updated
- [ ] Production build succeeds (npm run build)
- [ ] Package created (npx vsce package)
- [ ] Tested locally (code --install-extension)
- [ ] All features work correctly
- [ ] No console errors
- [ ] Git commit created
- [ ] Git tag created
- [ ] Pushed to GitHub
- [ ] GitHub release created (if applicable)