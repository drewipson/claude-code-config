---
name: Cross-Platform File Operations
description: Handle file operations correctly across macOS, Windows, and Linux
version: 1.0.0
---

# Cross-Platform File Operations Skill

This skill provides expertise in handling file system operations that work correctly across all major operating systems: macOS, Windows, and Linux.

## Platform Differences

### Path Separators
- **Windows**: Uses backslash `\` (e.g., `C:\Users\name\file.txt`)
- **macOS/Linux**: Uses forward slash `/` (e.g., `/Users/name/file.txt`)

### Home Directory
- **Windows**: `C:\Users\username` (via `%USERPROFILE%`)
- **macOS**: `/Users/username` (via `$HOME`)
- **Linux**: `/home/username` (via `$HOME`)

### Line Endings
- **Windows**: CRLF (`\r\n`)
- **macOS/Linux**: LF (`\n`)

### File System Case Sensitivity
- **Windows**: Case-insensitive (file.txt = FILE.TXT)
- **macOS**: Case-insensitive by default (configurable to case-sensitive)
- **Linux**: Case-sensitive (file.txt ≠ FILE.TXT)

## Safe Path Operations

### Always Use `path` Module

```typescript
import * as path from 'path';

// Good - works everywhere
const configPath = path.join(homeDir, '.claude', 'settings.json');

// Bad - breaks on Windows
const configPath = `${homeDir}/.claude/settings.json`;
```

### Path Methods

```typescript
// Join path segments
path.join('a', 'b', 'c')  // 'a/b/c' on Unix, 'a\b\c' on Windows

// Normalize path (resolve '..' and '.')
path.normalize('/foo/bar/../baz')  // '/foo/baz'

// Get directory name
path.dirname('/foo/bar/file.txt')  // '/foo/bar'

// Get file name
path.basename('/foo/bar/file.txt')  // 'file.txt'

// Get file extension
path.extname('file.txt')  // '.txt'

// Parse path into components
path.parse('/foo/bar/file.txt')
// {
//   root: '/',
//   dir: '/foo/bar',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

// Resolve absolute path
path.resolve('foo', 'bar', 'baz')  // Absolute path from cwd

// Check if path is absolute
path.isAbsolute('/foo/bar')  // true
path.isAbsolute('foo/bar')   // false
```

## Home Directory Resolution

```typescript
import * as os from 'os';

/**
 * Get user home directory (cross-platform)
 */
function getHomeDirectory(): string {
  // Preferred method (Node.js built-in)
  return os.homedir();

  // Alternative using environment variables
  // return process.env.HOME || process.env.USERPROFILE || os.homedir();
}

/**
 * Get Claude global config directory
 */
function getClaudeGlobalPath(): string {
  return path.join(getHomeDirectory(), '.claude');
}

// Usage
const globalSettingsPath = path.join(getClaudeGlobalPath(), 'settings.json');
```

## Async File Operations

**Always use async** to avoid blocking the extension host:

```typescript
import * as fs from 'fs/promises';

// Read file
async function readFile(filePath: string): Promise<string> {
  try {
    return await fs.readFile(filePath, 'utf-8');
  } catch (error) {
    if ((error as NodeJS.ErrnoException).code === 'ENOENT') {
      throw new Error(`File not found: ${filePath}`);
    }
    throw error;
  }
}

// Write file (create directories if needed)
async function writeFile(filePath: string, content: string): Promise<void> {
  const dir = path.dirname(filePath);
  await fs.mkdir(dir, { recursive: true });
  await fs.writeFile(filePath, content, 'utf-8');
}

// Check if file exists
async function fileExists(filePath: string): Promise<boolean> {
  try {
    await fs.access(filePath);
    return true;
  } catch {
    return false;
  }
}

// Check if path is directory
async function isDirectory(dirPath: string): Promise<boolean> {
  try {
    const stats = await fs.stat(dirPath);
    return stats.isDirectory();
  } catch {
    return false;
  }
}

// Delete file
async function deleteFile(filePath: string): Promise<void> {
  try {
    await fs.unlink(filePath);
  } catch (error) {
    if ((error as NodeJS.ErrnoException).code !== 'ENOENT') {
      throw error;
    }
    // File doesn't exist, ignore
  }
}

// Delete directory recursively
async function deleteDirectory(dirPath: string): Promise<void> {
  await fs.rm(dirPath, { recursive: true, force: true });
}
```

## Directory Operations

```typescript
// List directory contents
async function listDirectory(dirPath: string): Promise<string[]> {
  const entries = await fs.readdir(dirPath);
  return entries;
}

// List with file info
async function listDirectoryDetailed(dirPath: string): Promise<fs.Dirent[]> {
  const entries = await fs.readdir(dirPath, { withFileTypes: true });
  return entries;
}

// Recursive file discovery
async function findFiles(
  dirPath: string,
  pattern: RegExp
): Promise<string[]> {
  const results: string[] = [];

  async function scan(currentPath: string) {
    try {
      const entries = await fs.readdir(currentPath, { withFileTypes: true });

      for (const entry of entries) {
        const fullPath = path.join(currentPath, entry.name);

        if (entry.isDirectory()) {
          await scan(fullPath);
        } else if (entry.isFile() && pattern.test(entry.name)) {
          results.push(fullPath);
        }
      }
    } catch (error) {
      // Handle permission errors gracefully
      console.warn(`Cannot access ${currentPath}:`, error);
    }
  }

  await scan(dirPath);
  return results;
}

// Usage
const mdFiles = await findFiles(claudePath, /\.md$/);
```

## VS Code URI

When working with VS Code APIs, use `vscode.Uri`:

```typescript
import * as vscode from 'vscode';

// Create URI from file path
const fileUri = vscode.Uri.file(filePath);

// Open file in editor
await vscode.workspace.openTextDocument(fileUri);

// Get file path from URI
const filePath = fileUri.fsPath;

// Parse URI
const uri = vscode.Uri.parse('file:///Users/name/file.txt');

// Join URI paths
const uri = vscode.Uri.joinPath(baseUri, 'subdir', 'file.txt');
```

## File System Watcher

```typescript
import * as vscode from 'vscode';

function createFileWatcher(
  workspaceFolder: vscode.WorkspaceFolder,
  pattern: string
): vscode.FileSystemWatcher {
  // Use RelativePattern for cross-platform support
  const watcher = vscode.workspace.createFileSystemWatcher(
    new vscode.RelativePattern(workspaceFolder, pattern)
  );

  watcher.onDidCreate(uri => {
    console.log('File created:', uri.fsPath);
  });

  watcher.onDidChange(uri => {
    console.log('File changed:', uri.fsPath);
  });

  watcher.onDidDelete(uri => {
    console.log('File deleted:', uri.fsPath);
  });

  return watcher;
}

// Usage
const watcher = createFileWatcher(workspaceFolder, '.claude/**/*');
context.subscriptions.push(watcher);
```

## File Permissions

```typescript
// Check if file is readable
async function isReadable(filePath: string): Promise<boolean> {
  try {
    await fs.access(filePath, fs.constants.R_OK);
    return true;
  } catch {
    return false;
  }
}

// Check if file is writable
async function isWritable(filePath: string): Promise<boolean> {
  try {
    await fs.access(filePath, fs.constants.W_OK);
    return true;
  } catch {
    return false;
  }
}

// Get file stats
async function getFileStats(filePath: string): Promise<fs.Stats> {
  return await fs.stat(filePath);
}

// Example usage
const stats = await getFileStats(filePath);
console.log('Size:', stats.size);
console.log('Created:', stats.birthtime);
console.log('Modified:', stats.mtime);
console.log('Is directory:', stats.isDirectory());
console.log('Is file:', stats.isFile());
```

## Error Handling

```typescript
async function safeFileOperation<T>(
  operation: () => Promise<T>,
  errorContext: string
): Promise<T | null> {
  try {
    return await operation();
  } catch (error) {
    const err = error as NodeJS.ErrnoException;

    switch (err.code) {
      case 'ENOENT':
        console.error(`${errorContext}: File or directory not found`);
        break;
      case 'EACCES':
        console.error(`${errorContext}: Permission denied`);
        break;
      case 'EEXIST':
        console.error(`${errorContext}: File already exists`);
        break;
      case 'ENOTDIR':
        console.error(`${errorContext}: Not a directory`);
        break;
      case 'EISDIR':
        console.error(`${errorContext}: Is a directory`);
        break;
      default:
        console.error(`${errorContext}:`, err.message);
    }

    return null;
  }
}

// Usage
const content = await safeFileOperation(
  () => readFile(configPath),
  'Failed to read config'
);
```

## Temporary Files

```typescript
import * as os from 'os';

/**
 * Get system temp directory
 */
function getTempDir(): string {
  return os.tmpdir();
}

/**
 * Create temp file path
 */
function createTempFilePath(prefix: string, extension: string): string {
  const timestamp = Date.now();
  const random = Math.random().toString(36).substring(7);
  const filename = `${prefix}-${timestamp}-${random}.${extension}`;
  return path.join(getTempDir(), filename);
}

// Usage
const tempFile = createTempFilePath('claude-export', 'json');
await writeFile(tempFile, JSON.stringify(data));

// Clean up after use
await deleteFile(tempFile);
```

## Glob Patterns

For pattern matching, use VS Code's built-in `RelativePattern`:

```typescript
// Find all markdown files in .claude directory
const files = await vscode.workspace.findFiles(
  new vscode.RelativePattern(workspaceFolder, '.claude/**/*.md'),
  '**/node_modules/**'  // Exclude node_modules
);

// Convert URIs to file paths
const filePaths = files.map(uri => uri.fsPath);
```

## File Name Validation

```typescript
/**
 * Validate file name is safe for all platforms
 */
function isValidFileName(name: string): boolean {
  // Invalid characters on Windows: < > : " / \ | ? *
  // Also exclude control characters and leading/trailing spaces
  const invalidChars = /[<>:"/\\|?*\x00-\x1f]/;

  if (invalidChars.test(name)) {
    return false;
  }

  // Check for reserved names on Windows
  const reserved = /^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])(\.|$)/i;
  if (reserved.test(name)) {
    return false;
  }

  // Check length (255 chars max on most systems)
  if (name.length === 0 || name.length > 255) {
    return false;
  }

  // No leading/trailing spaces or periods
  if (name.trim() !== name || name.endsWith('.')) {
    return false;
  }

  return true;
}

/**
 * Sanitize file name for cross-platform use
 */
function sanitizeFileName(name: string): string {
  return name
    .replace(/[<>:"/\\|?*\x00-\x1f]/g, '-')  // Replace invalid chars
    .replace(/^\.+/, '')                      // Remove leading dots
    .replace(/\.+$/, '')                      // Remove trailing dots
    .trim()                                   // Remove leading/trailing spaces
    .substring(0, 255);                       // Limit length
}
```

## Common Patterns

### Move File

```typescript
async function moveFile(
  sourcePath: string,
  destPath: string
): Promise<void> {
  // Ensure destination directory exists
  const destDir = path.dirname(destPath);
  await fs.mkdir(destDir, { recursive: true });

  // Move file
  await fs.rename(sourcePath, destPath);
}
```

### Copy File

```typescript
async function copyFile(
  sourcePath: string,
  destPath: string
): Promise<void> {
  // Ensure destination directory exists
  const destDir = path.dirname(destPath);
  await fs.mkdir(destDir, { recursive: true });

  // Copy file
  await fs.copyFile(sourcePath, destPath);
}
```

### Ensure Directory Exists

```typescript
async function ensureDir(dirPath: string): Promise<void> {
  await fs.mkdir(dirPath, { recursive: true });
}
```

## Testing Cross-Platform

```typescript
// Test on different platforms
describe('File Operations', () => {
  it('should work on Windows paths', () => {
    const result = path.join('C:', 'Users', 'name', 'file.txt');
    // On Windows: C:\Users\name\file.txt
    // On Unix: C:/Users/name/file.txt (still valid)
  });

  it('should work on Unix paths', () => {
    const result = path.join('/', 'home', 'name', 'file.txt');
    // On Unix: /home/name/file.txt
    // On Windows: \home\name\file.txt (or C:\home\name\file.txt)
  });

  it('should normalize paths', () => {
    const result = path.normalize('foo/bar/../baz');
    // Both: foo/baz (or foo\baz on Windows)
  });
});
```

## Best Practices

1. **Always use `path.join()`** - Never concatenate paths with string literals
2. **Use async file operations** - Never use sync methods (readFileSync, etc.)
3. **Handle errors gracefully** - Check error codes (ENOENT, EACCES, etc.)
4. **Validate file names** - Remove invalid characters before creating files
5. **Use VS Code URIs** - When working with VS Code APIs, use vscode.Uri
6. **Test on all platforms** - If possible, test on Windows, macOS, and Linux
7. **Use relative patterns** - For file watchers and glob patterns
8. **Respect case sensitivity** - Don't assume case-insensitive file systems
9. **Create directories recursively** - Use `{ recursive: true }` option
10. **Clean up temp files** - Delete temporary files when done

## Usage

This skill provides:
- Cross-platform path operations
- Safe file system access
- Directory traversal and discovery
- File validation and sanitization
- VS Code URI integration
- Error handling patterns
- Async file operations