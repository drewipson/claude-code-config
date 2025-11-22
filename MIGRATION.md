# Migration Guide

## Version 1.0.0 - Package Rename to Claude Code Config

### ⚠️ Breaking Changes

This version includes a major package rename from `claude-code-manager` to `claude-code-config`. This change affects the VS Code configuration namespace.

---

## What Changed

### Package Identifiers
- **Package Name**: `claude-code-manager` → `claude-code-config`
- **Extension ID**: `claude-code-manager` → `claude-code-config`
- **Publisher**: Updated to official publisher ID

### Configuration Namespace
**This is the breaking change that requires action from users.**

The VS Code settings namespace has changed:
- **Old**: `claudeCodeManager.*`
- **New**: `claudeCodeConfig.*`

---

## Required Migration Steps

### Step 1: Update VS Code Settings

If you have customized any Claude Code Config settings, you need to update them in your VS Code settings.

#### For User Settings (Global)

1. Open VS Code Settings (UI): `Cmd+,` (Mac) or `Ctrl+,` (Windows/Linux)
2. Search for `claudeCodeManager`
3. If you find any settings, note their values
4. Delete the old `claudeCodeManager.*` settings
5. Add the new `claudeCodeConfig.*` settings with the same values

**OR**

1. Open settings.json: `Cmd+Shift+P` → "Preferences: Open Settings (JSON)"
2. Find and replace:

```json
// OLD SETTINGS (remove these)
{
  "claudeCodeManager.autoRefresh": true,
  "claudeCodeManager.globalClaudePath": "/custom/path/.claude"
}
```

```json
// NEW SETTINGS (add these)
{
  "claudeCodeConfig.autoRefresh": true,
  "claudeCodeConfig.globalClaudePath": "/custom/path/.claude"
}
```

#### For Workspace Settings (Project-specific)

If you have workspace settings in `.vscode/settings.json`:

1. Open `.vscode/settings.json` in your project
2. Replace all instances of `claudeCodeManager` with `claudeCodeConfig`

### Step 2: Reinstall Extension (If Necessary)

If you installed from VSIX or have issues:

1. Uninstall old version: `Extensions` → Search "Claude Code" → Click gear → Uninstall
2. Install new version from Marketplace or VSIX
3. Reload VS Code

---

## Configuration Reference

### Available Settings

| Old Setting | New Setting | Description |
|------------|-------------|-------------|
| `claudeCodeManager.autoRefresh` | `claudeCodeConfig.autoRefresh` | Auto-refresh views on file changes (default: `true`) |
| `claudeCodeManager.globalClaudePath` | `claudeCodeConfig.globalClaudePath` | Custom global .claude directory (default: `~/.claude`) |

### Migration Example

**Before (v0.x):**
```json
{
  "claudeCodeManager.autoRefresh": false,
  "claudeCodeManager.globalClaudePath": "/Users/me/custom-claude"
}
```

**After (v1.0.0+):**
```json
{
  "claudeCodeConfig.autoRefresh": false,
  "claudeCodeConfig.globalClaudePath": "/Users/me/custom-claude"
}
```

---

## What Stays the Same

### No Changes Required For:
- ✅ Your Claude Code configuration files (`.claude/`, `~/.claude/`)
- ✅ CLAUDE.md memory files
- ✅ Slash commands
- ✅ Skills and sub-agents
- ✅ Hooks and permissions
- ✅ All extension functionality and features

### File Structure
The extension still manages the same files and locations:
```
~/.claude/                  # Global configs (unchanged)
├── CLAUDE.md
├── commands/
├── skills/
├── agents/
└── settings.json

.claude/                    # Project configs (unchanged)
├── CLAUDE.md
├── commands/
├── agents/
└── settings.local.json
```

---

## Troubleshooting

### Extension Not Loading

**Problem**: Extension doesn't appear after update.

**Solution**:
1. Restart VS Code completely
2. Check Extensions view for "Claude Code Config"
3. If missing, reinstall from Marketplace

### Settings Not Working

**Problem**: Custom settings are not being applied.

**Solution**:
1. Verify you've updated the namespace from `claudeCodeManager` to `claudeCodeConfig`
2. Check for typos in setting names
3. Restart VS Code after changing settings

### Configuration Path Not Found

**Problem**: Extension can't find your custom `.claude` directory.

**Solution**:
1. Update the setting using the new namespace:
   ```json
   {
     "claudeCodeConfig.globalClaudePath": "/your/custom/path/.claude"
   }
   ```
2. Ensure the path exists and has correct permissions
3. Restart VS Code

### Old and New Settings Conflict

**Problem**: Both old and new settings exist in settings.json.

**Solution**:
1. Remove ALL `claudeCodeManager.*` settings
2. Keep only `claudeCodeConfig.*` settings
3. Restart VS Code

---

## Why This Change?

This rename provides:
- **Consistency**: Package name matches display name ("Claude Code Config")
- **Clarity**: More descriptive and searchable name
- **Future-Proofing**: Better namespace for long-term development

---

## Need Help?

If you encounter issues during migration:

1. **Check this guide** - Most common issues are covered above
2. **GitHub Issues** - [Report a problem](https://github.com/your-username/claude-code-config/issues)
3. **Discussions** - [Ask the community](https://github.com/your-username/claude-code-config/discussions)

---

## Quick Migration Checklist

- [ ] Backup your VS Code settings (optional but recommended)
- [ ] Update user settings: `claudeCodeManager.*` → `claudeCodeConfig.*`
- [ ] Update workspace settings (if any): `claudeCodeManager.*` → `claudeCodeConfig.*`
- [ ] Remove old `claudeCodeManager.*` settings
- [ ] Restart VS Code
- [ ] Verify extension loads correctly
- [ ] Test that your custom settings are working

---

**Thank you for using Claude Code Config!** We appreciate your patience with this one-time breaking change.