# Commit Changes

Create a conventional commit with staged changes.

## Instructions

1. Review staged files with `git status`
2. Review the actual changes with `git diff --cached`
3. Analyze the changes and determine the appropriate conventional commit type:
   - **feat:** New feature or enhancement
   - **fix:** Bug fix
   - **docs:** Documentation changes only
   - **style:** Code style changes (formatting, whitespace, no logic changes)
   - **refactor:** Code refactoring (no feature changes or fixes)
   - **test:** Adding or updating tests
   - **chore:** Build process, dependencies, tooling
   - **perf:** Performance improvements
4. Write a clear, concise commit message following this format:
   ```
   <type>: <description>

   [optional body explaining the change]

   [optional footer with breaking changes or issue references]
   ```
5. The description should:
   - Be lowercase (except proper nouns)
   - Be concise (50 characters or less)
   - Not end with a period
   - Focus on what changed, not how or why
6. Create the commit using:
   ```bash
   git commit -m "type: description"
   ```

## Conventional Commit Types

| Type | Description | Example |
|------|-------------|---------|
| **feat** | New feature | `feat: add hook creation wizard` |
| **fix** | Bug fix | `fix: resolve file watcher crash on Windows` |
| **docs** | Documentation | `docs: update CLAUDE.md with testing guide` |
| **style** | Formatting | `style: apply prettier formatting` |
| **refactor** | Code refactoring | `refactor: extract file discovery to service` |
| **test** | Tests | `test: add unit tests for YAML parser` |
| **chore** | Tooling/deps | `chore: update esbuild to 0.20.0` |
| **perf** | Performance | `perf: optimize tree view refresh` |

## Examples

### Good Commit Messages
```bash
feat: add color support for sub-agents
fix: handle missing YAML frontmatter gracefully
docs: add VS Code extension development guide
refactor: simplify hook JSON manipulation
test: add file discovery service tests
chore: update dependencies
```

### Bad Commit Messages
```bash
Updated files  # Not descriptive, no type
Fixed bug.     # Too vague, has period
WIP            # Work in progress commits should be avoided
feat: Added new feature for creating hooks with visual wizard UI  # Too long
```

## Commit Body (Optional)

Add a body for complex changes:
```bash
git commit -m "feat: add visual hook builder

Implements a multi-step wizard for creating Claude Code hooks
without manually editing JSON. Supports all event types and
both command-based and prompt-based hooks.

Includes validation and user-friendly error messages."
```

## Output

After committing, show:
- Commit hash
- Files changed
- Lines added/removed
- Next steps (e.g., "Ready to push" or "Continue development")

## Never Commit

- Directly to main branch (create a feature branch)
- Broken code or failing tests
- Debugging statements (console.log, debugger)
- Sensitive information (API keys, tokens, passwords)
- Large binary files without .gitattributes
- Unrelated changes (split into multiple commits)