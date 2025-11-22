# Claude Code Config - VS Code Extension

**A unified management interface for Claude Code configurations**

This project is a VS Code extension that provides centralized management for Claude Code settings including memories (CLAUDE.md files), slash commands, skills, sub-agents, permissions, and hooks.

---

## Project Overview

### Purpose

Claude Code Config solves the problem of scattered configuration files by creating a single command center for managing all Claude Code settings. Instead of navigating between `~/.claude/` and `.claude/` directories, users can visualize, create, edit, and organize everything from a VS Code sidebar.

### Key Capabilities

- **Unified View**: All Claude Code configurations in one tree view interface
- **Scope Management**: Move configs between global (~/.claude/) and project (.claude/) with one click
- **Visual Hook Builder**: Create automation hooks without touching JSON
- **Folder Organization**: Group commands and sub-agents logically
- **Live Sync**: Auto-refresh when configuration files change
- **Color-Coded Agents**: Sub-agents display in configured colors

---

## Tech Stack

### Core Technologies

- **Language**: TypeScript (ES2022, strict mode enabled)
- **Build Tool**: esbuild (fast bundler with watch mode)
- **Runtime**: Node.js 20+
- **Platform**: VS Code Extension API 1.85.0+
- **Module Format**: CommonJS (required for VS Code extensions)

### Key Dependencies

- `chart.js ^4.4.1` - For analytics dashboard features
- `vscode` (external) - VS Code Extension API (never bundled)

### Development Tools

- ESLint with TypeScript parser
- TypeScript 5.3.2 compiler
- VS Code Extension Test framework
- esbuild for fast incremental builds

---

## Architecture

### Directory Structure

```
src/
├── extension.ts              # Main entry point, activation, command registration
├── core/
│   ├── types.ts             # All TypeScript interfaces and type definitions
│   └── constants.ts         # Shared constants, IDs, paths, and configs
├── providers/
│   └── claudeTreeDataProvider.ts  # All 8 tree view provider implementations
├── services/
│   ├── fileDiscoveryService.ts    # Discovers Claude config files across scopes
│   ├── fileOperationsService.ts   # CRUD operations for files and folders
│   ├── hooksService.ts            # Hooks JSON manipulation and management
│   ├── permissionsService.ts      # Permission rule parsing and discovery
│   └── mcpService.ts              # MCP server configuration discovery
└── utils/
    ├── yamlParser.ts        # YAML frontmatter extraction from markdown
    └── markdownParser.ts    # Markdown heading and section parsing

resources/
└── templates/               # File templates for quick creation
    ├── CLAUDE.md.template
    ├── SKILL.md.template
    └── command.md.template
```

### Key Architectural Patterns

#### 1. Service Layer Pattern

Business logic is separated into specialized services:

- **FileDiscoveryService**: Recursively scans and categorizes Claude config files
- **FileOperationsService**: Handles all file/folder CRUD operations
- **HooksService**: Manages hook JSON structures and validation
- **PermissionsService**: Parses permission rules from settings files
- **McpService**: Discovers and manages MCP server configurations

#### 2. Provider Pattern

VS Code TreeDataProviders for the 8 sidebar sections:

- MemoriesTreeProvider (CLAUDE.md files with sections)
- CommandsTreeProvider (slash command .md files)
- SkillsTreeProvider (SKILL.md in folders)
- SubAgentsTreeProvider (agent .md files with colors)
- PermissionsTreeProvider (permission rules visualization)
- HooksTreeProvider (hook automation organized by event)
- DocumentationTreeProvider (links to Claude Code docs)

#### 3. Command Pattern

21+ registered VS Code commands handle all user actions:

- File operations: open, rename, delete, move, copy path
- Creation: memory, command, skill, sub-agent, folder, hook
- Navigation: reveal in finder, go to specific line
- Hooks: create, edit, delete, duplicate, copy JSON

#### 4. Type Safety First

Comprehensive TypeScript interfaces in `core/types.ts`:

- ClaudeFile, ClaudeTreeItem, ClaudeScope
- Hook types, matchers, and event configurations
- Permission rules and patterns
- Service method signatures

---

## Code Style & Conventions

### Formatting

- **Indentation**: 2 spaces (no tabs)
- **Line Length**: 100 characters maximum
- **Quotes**: Single quotes for strings
- **Semicolons**: Always use semicolons
- **Trailing Commas**: Use in multi-line structures

### Naming Conventions

- **Variables**: camelCase (`fileDiscoveryService`, `treeItems`)
- **Functions**: camelCase (`createHook`, `discoverFiles`)
- **Classes**: PascalCase (`FileDiscoveryService`, `HooksService`)
- **Interfaces**: PascalCase (`ClaudeFile`, `HookConfiguration`)
- **Constants**: SCREAMING_SNAKE_CASE (`CLAUDE_GLOBAL_PATH`)
- **Files**: kebab-case (`file-discovery-service.ts`)

### Import Order

1. External dependencies (vscode, path, fs)
2. Internal core modules (types, constants)
3. Services
4. Utilities
5. Type-only imports (use `import type`)

Example:

```typescript
import * as vscode from "vscode";
import * as path from "path";
import * as fs from "fs/promises";

import { ClaudeFile, ClaudeScope } from "./core/types";
import { CLAUDE_GLOBAL_PATH } from "./core/constants";

import { FileDiscoveryService } from "./services/fileDiscoveryService";
import { yamlParser } from "./utils/yamlParser";

import type { HookConfiguration } from "./core/types";
```

---

## VS Code Extension Specifics

### Extension Activation

- **Activation Event**: `onStartupFinished` (activates after VS Code loads)
- **Entry Point**: `src/extension.ts` → `activate()` function
- **Deactivation**: `deactivate()` function for cleanup

### Command Registration

All commands must be:

1. Defined in `package.json` under `contributes.commands`
2. Registered in `activate()` using `vscode.commands.registerCommand`
3. Added to context subscriptions for proper disposal

Example:

```typescript
const disposable = vscode.commands.registerCommand(
  "claudeCodeManager.openFile",
  async (item: ClaudeTreeItem) => {
    // Command implementation
  }
);
context.subscriptions.push(disposable);
```

### TreeDataProvider Implementation

Key methods to implement:

- `getTreeItem(element)`: Convert data to TreeItem
- `getChildren(element?)`: Return child elements
- `refresh()`: Trigger tree view update with `_onDidChangeTreeData.fire()`

Use `vscode.EventEmitter` for change notifications:

```typescript
private _onDidChangeTreeData = new vscode.EventEmitter<ClaudeTreeItem | undefined>();
readonly onDidChangeTreeData = this._onDidChangeTreeData.event;

refresh(): void {
  this._onDidChangeTreeData.fire(undefined);
}
```

### File System Operations

**Always use async/await** for file operations:

```typescript
import * as fs from "fs/promises";

// Good
await fs.readFile(filePath, "utf-8");
await fs.mkdir(dirPath, { recursive: true });

// Bad - sync operations block the extension host
fs.readFileSync(filePath); // DON'T USE
```

### Cross-Platform Path Handling

Use `vscode.Uri` and Node's `path` module:

```typescript
import * as path from "path";
import * as vscode from "vscode";

// Get home directory
const homeDir = process.env.HOME || process.env.USERPROFILE;

// Join paths safely
const configPath = path.join(homeDir, ".claude", "settings.json");

// Use Uri for VS Code APIs
const fileUri = vscode.Uri.file(configPath);
await vscode.workspace.openTextDocument(fileUri);
```

### External Module Handling

**CRITICAL**: Never bundle the `vscode` module

- Configure in `esbuild.mjs`: `external: ['vscode']`
- VS Code provides the vscode module at runtime
- Bundling it causes extension load failures

---

## Key Development Patterns

### 1. File Discovery Pattern

Recursively scan directories to find Claude config files:

```typescript
async function discoverFiles(
  dirPath: string,
  scope: ClaudeScope
): Promise<ClaudeFile[]> {
  const files: ClaudeFile[] = [];
  try {
    const entries = await fs.readdir(dirPath, { withFileTypes: true });
    for (const entry of entries) {
      if (entry.isDirectory()) {
        // Recurse into subdirectories
        const subFiles = await discoverFiles(
          path.join(dirPath, entry.name),
          scope
        );
        files.push(...subFiles);
      } else if (entry.name.endsWith(".md")) {
        // Process markdown files
        files.push({
          name: entry.name,
          path: path.join(dirPath, entry.name),
          scope,
        });
      }
    }
  } catch (error) {
    // Handle errors gracefully
  }
  return files;
}
```

### 2. YAML Frontmatter Parsing

Skills and agents use YAML frontmatter for metadata:

```typescript
// Parse frontmatter
const content = await fs.readFile(filePath, "utf-8");
const frontmatter = yamlParser.extractFrontmatter(content);

// Access metadata
const color = frontmatter?.color || "default";
const description = frontmatter?.description || "";
```

### 3. Hook JSON Manipulation

Hooks are stored in settings.json as nested structures:

```typescript
interface HookConfiguration {
  event: string;
  matcher?: {
    type: string;
    pattern: string;
  };
  command?: string;
  prompt?: string;
}

// Add hook to settings
const settings = JSON.parse(await fs.readFile(settingsPath, "utf-8"));
if (!settings.hooks) settings.hooks = [];
settings.hooks.push(newHook);
await fs.writeFile(settingsPath, JSON.stringify(settings, null, 2));
```

### 4. Tree Item with Commands

Attach commands to tree items for click actions:

```typescript
const treeItem = new vscode.TreeItem(
  label,
  vscode.TreeItemCollapsibleState.None
);
treeItem.command = {
  command: "claudeCodeManager.openFile",
  title: "Open File",
  arguments: [fileData],
};
return treeItem;
```

### 5. File Watcher for Live Updates

Watch .claude directories for changes:

```typescript
const watcher = vscode.workspace.createFileSystemWatcher(
  new vscode.RelativePattern(workspaceFolder, ".claude/**/*")
);

watcher.onDidCreate(() => treeProvider.refresh());
watcher.onDidChange(() => treeProvider.refresh());
watcher.onDidDelete(() => treeProvider.refresh());

context.subscriptions.push(watcher);
```

---

## Common Development Tasks

### Adding a New Tree View Section

1. Define tree item interface in `src/core/types.ts`
2. Create provider class in `src/providers/claudeTreeDataProvider.ts`
3. Implement `getTreeItem()` and `getChildren()` methods
4. Register view in `package.json` under `contributes.views`
5. Register provider in `src/extension.ts` activate function
6. Add refresh command and register it

### Creating a New Service

1. Create file in `src/services/` with descriptive name
2. Export a class with public methods for the service's domain
3. Use dependency injection for other services if needed
4. Make all I/O operations async
5. Handle errors gracefully and return meaningful error messages
6. Import and use in `extension.ts` or providers

### Registering a New Command

1. Add command definition to `package.json`:

```json
{
  "command": "claudeCodeManager.myCommand",
  "title": "My Command",
  "category": "Claude Code Config"
}
```

2. Register handler in `src/extension.ts`:

```typescript
context.subscriptions.push(
  vscode.commands.registerCommand("claudeCodeManager.myCommand", async () => {
    // Implementation
  })
);
```

### Adding a File Template

1. Create template file in `resources/templates/`
2. Use placeholders for dynamic content
3. Load template in FileOperationsService
4. Replace placeholders when creating new files

---

## Testing Strategy

### Extension Development Host

- Press **F5** in VS Code to launch Extension Development Host
- New VS Code window opens with extension loaded
- Set breakpoints in TypeScript for debugging
- Console output appears in original VS Code Debug Console

### Manual Testing Checklist

- [ ] File discovery works for global and project scopes
- [ ] Tree views display correct hierarchies
- [ ] File operations (create, rename, delete, move) work correctly
- [ ] Hooks can be created, edited, and deleted
- [ ] Permissions are parsed and displayed accurately
- [ ] Color-coded agents display properly
- [ ] File watcher triggers refresh on changes
- [ ] Cross-platform paths work (test on macOS/Windows/Linux)

### Unit Testing

(To be implemented)

- Mock file system with `memfs` or similar
- Test services in isolation
- Validate YAML and markdown parsing
- Test hook JSON manipulation

### Integration Testing

- Use VS Code Extension Test framework
- Test end-to-end workflows
- Validate VS Code API interactions

---

## Build & Development Commands

### Development Workflow

```bash
# Install dependencies
npm install

# Development mode with auto-rebuild (recommended)
npm run watch

# Production build (minified)
npm run build

# Run ESLint
npm run lint

# Launch Extension Development Host
# Press F5 in VS Code
```

### Build Configuration (esbuild)

Configuration in `esbuild.mjs`:

- **Entry**: `src/extension.ts`
- **Output**: `dist/extension.js`
- **Bundle**: true (all dependencies bundled except vscode)
- **Format**: CommonJS (required for VS Code)
- **External**: `vscode` module (provided by VS Code)
- **Minification**: Production builds only
- **Source Maps**: Development builds for debugging

### Packaging for Distribution

```bash
# Create .vsix package
npx vsce package

# Install locally
code --install-extension claude-code-manager-0.0.1.vsix
```

---

## Guidelines

### Do's ✅

- Always use async/await for file operations
- Use TypeScript strict mode and avoid `any` types
- Handle errors gracefully with try/catch
- Use vscode.Uri for file paths in VS Code APIs
- Test in Extension Development Host before committing
- Use conventional commits (feat:, fix:, docs:, refactor:)
- Update package.json version when releasing
- Keep tree provider logic separate from service logic
- Use VSCode's built-in icons (ThemeIcon) when possible
- Make all file operations cross-platform compatible

### Don'ts ❌

- **Never** bundle the vscode module in esbuild
- **Never** use synchronous file operations (fs.readFileSync)
- **Never** commit directly to main branch
- **Never** use `any` type without strong justification
- **Never** leave console.log statements in production code
- **Never** skip error handling in async functions
- **Never** hardcode paths (use path.join and os.homedir)
- **Never** mutate tree items after returning them
- **Never** assume file system structure (always check existence)
- **Never** ignore extension activation errors

---

## Known Issues & Quirks

### VS Code Extension Constraints

1. **External CSS**: Cannot load external stylesheets. Use VS Code's ThemeIcon and ThemeColor APIs
2. **Bundling**: Must use CommonJS format, not ESM
3. **Activation**: Heavy operations should be deferred until after activation
4. **File Watchers**: Limited to specific glob patterns, not entire file system

### Project-Specific Notes

- **YAML Parsing**: Using custom lightweight parser, not a full YAML library
- **Frontmatter**: Only parses top-level key-value pairs, not nested structures
- **Hook Events**: 10 event types supported (see hooks documentation)
- **Permission Patterns**: Supports wildcards but not regex

---

## Debugging Tips

### Common Issues

**Extension doesn't activate**

- Check activation events in package.json
- Look for errors in Debug Console (original VS Code window)
- Verify all dependencies are installed

**Tree view not showing items**

- Add console.log in getChildren() to verify it's called
- Check if file discovery is returning results
- Verify file paths are absolute, not relative

**File operations failing**

- Check file permissions
- Verify directory exists before creating files
- Use try/catch and log errors
- Test on different operating systems

**Hooks not working**

- Validate JSON structure in settings.local.json
- Check event type spelling (case-sensitive)
- Verify matcher patterns match expected format

---

## Resources & References

### Official Documentation

- [VS Code Extension API](https://code.visualstudio.com/api)
- [TreeDataProvider Guide](https://code.visualstudio.com/api/extension-guides/tree-view)
- [VS Code Extension Samples](https://github.com/microsoft/vscode-extension-samples)
- [Claude Code Documentation](https://code.claude.com/docs/en/overview)

### Internal Documentation

- README.md - User-facing feature documentation
- package.json - Extension manifest and configuration
- CHANGELOG.md - Version history and changes

---

## Future Enhancements

### Areas for Improvement

- Performance optimization for large config sets
- Better error messages and user feedback
- Undo/redo support for file operations
- Search and filter capabilities
- Keyboard shortcuts for common actions

---

## Contributing

When contributing to this project:

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/amazing-feature`)
3. Follow the code style guidelines above
4. Test thoroughly in Extension Development Host
5. Use conventional commits (`feat:`, `fix:`, `docs:`, etc.)
6. Update documentation if adding features
7. Submit a pull request with clear description

---

**Built with ❤️ for the Claude Code community**

This extension aims to make Claude Code configuration management effortless. If you have ideas or feedback, open an issue on GitHub!
