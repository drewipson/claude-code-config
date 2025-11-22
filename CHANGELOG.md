# Changelog

All notable changes to the Claude Code Config extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-01-XX

### 🎉 Major Release

This is the first major release of Claude Code Config with significant improvements and a breaking change for better consistency.

### ⚠️ BREAKING CHANGES

- **Package Renamed**: Extension package renamed from `claude-code-manager` to `claude-code-config`
- **Configuration Namespace Changed**: VS Code settings namespace changed from `claudeCodeManager.*` to `claudeCodeConfig.*`
  - Users must update their VS Code settings
  - See [MIGRATION.md](MIGRATION.md) for detailed migration guide
  - **Action Required**: Update `claudeCodeManager.autoRefresh` → `claudeCodeConfig.autoRefresh`
  - **Action Required**: Update `claudeCodeManager.globalClaudePath` → `claudeCodeConfig.globalClaudePath`

### ✨ Added

- **Move to Folder**: New functionality to move files and folders into other folders within the same scope
  - Right-click any file or folder → "Move to Folder"
  - Select target folder from quick pick menu
  - Supports conflict resolution (overwrite/rename/cancel)
  - Validates scope boundaries (cannot move across global/project scopes)
  - Context menu available for both files and folders

### 🐛 Fixed

- **Markdown Parser**: Code block headings no longer appear in dropdown navigation
  - Fixed issue where `# comments` inside ``` code blocks were treated as markdown headings
  - Parser now correctly tracks code block state and ignores headings within code blocks
  - Improves accuracy of CLAUDE.md, skills, and command section navigation

### 📝 Documentation

- Added comprehensive MIGRATION.md guide for breaking changes
- Updated README.md with new package name and configuration references
- Updated all internal documentation references

### 🔧 Changed

- Extension ID updated to `claude-code-config`
- View container ID updated for consistency
- All configuration references updated throughout codebase

---

## [0.1.0] - 2024-XX-XX

### Initial Release

- ✨ Unified configuration management interface
- 📁 Tree view for memories (CLAUDE.md files)
- 💻 Slash commands management
- 💡 Skills with YAML frontmatter support
- 🤖 Sub-agents with color coding
- 🔧 MCP servers visualization
- 🔒 Permissions parsing and display
- 🪝 Visual hook builder and management
- 📚 Documentation links panel
- 🔄 Auto-refresh on file changes
- 🌍 Global and project scope management
- 📂 Folder organization for commands and agents
- 🎨 Markdown section navigation
- 🛠️ File operations (create, rename, delete, move between scopes)

---

## Migration Guide

For users upgrading from v0.x to v1.0.0, please see [MIGRATION.md](MIGRATION.md) for detailed instructions on updating your configuration.