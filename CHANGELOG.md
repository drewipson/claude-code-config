# Changelog

All notable changes to the Claude Code Config extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1](https://github.com/drewipson/claude-code-config/compare/claude-code-config-v1.0.0...claude-code-config-v1.0.1) (2025-11-24)


### Bug Fixes

* resolve all ESLint errors and warnings ([25a80af](https://github.com/drewipson/claude-code-config/commit/25a80af1f61147d1e30fce4d16e41e92060ee7e9))
* resolve moveToFolder error and remove MCP servers section ([904c4cc](https://github.com/drewipson/claude-code-config/commit/904c4cc5c99cb8dffff9f48580038728390e4c40))


### Documentation

* enhance README with feature screenshots and fix typo ([7375342](https://github.com/drewipson/claude-code-config/commit/7375342ca620ac6ff5d1b49a31e6da5f9921b690))
* enhance README with feature screenshots and fix typo ([0e6995d](https://github.com/drewipson/claude-code-config/commit/0e6995d476d4bac644b5736f13d4dbd37b493dae))


### Continuous Integration

* add CI/CD pipeline for VS Code Marketplace publishing ([6446973](https://github.com/drewipson/claude-code-config/commit/6446973c9b2e37f816ce118a57ee4f9fcfd999bd))
* add GitHub Actions CI/CD pipeline ([39c3dfa](https://github.com/drewipson/claude-code-config/commit/39c3dfa4af80a6aeba80b05a05cb2d90b66d5504))
* add GitHub Actions workflows for CI/CD pipeline ([ca226e9](https://github.com/drewipson/claude-code-config/commit/ca226e9e706e652c3f651d71c9a087040b396416))
* simplify CI to lint and build only ([729fb1b](https://github.com/drewipson/claude-code-config/commit/729fb1bb701bf07454e70f7dcd6c5bc664a1f2f2))
* update to non-deprecated release-please action ([5e03c6c](https://github.com/drewipson/claude-code-config/commit/5e03c6ce5870bf72e618411f732a01cfbe8ab6b3))

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
