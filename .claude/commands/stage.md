# Stage Changes

Stage modified files for commit using git add.

## Instructions

1. Run `git status` to see all modified, untracked, and deleted files
2. Review the changes and identify which files should be staged
3. Stage relevant files using `git add <file>` for specific files or `git add .` for all changes
4. Exclude any files that shouldn't be committed:
   - Temporary files
   - Local configuration files (*.local.json)
   - Build artifacts (dist/, node_modules/)
   - IDE-specific files (.vscode/settings.json if not project-wide)
   - Files with sensitive information
5. Run `git status` again to confirm which files are staged
6. Show a summary of what will be committed

## Best Practices

- **Be selective**: Only stage files related to the current change
- **Review diffs**: Use `git diff` to verify changes before staging
- **Atomic commits**: Stage files that represent a single logical change
- **No debugging code**: Remove console.log and debugging statements before staging
- **Check .gitignore**: Ensure excluded files aren't being staged

## Example Output

```
Staged files for commit:
✓ src/extension.ts (modified)
✓ src/services/hooksService.ts (modified)
✓ CLAUDE.md (modified)

Ready to commit with /commit
```