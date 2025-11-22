# Create Draft Pull Request

Create a draft pull request for the current branch following project conventions.

## Instructions

1. **Verify prerequisites:**
   - Check current branch: `git branch --show-current`
   - Ensure commits exist: `git log origin/main..HEAD`
   - Confirm branch is pushed: `git push -u origin <branch-name>` if needed

2. **Analyze all changes:**
   - Review full diff: `git diff main...HEAD`
   - List all commits: `git log main..HEAD --oneline`
   - Understand the scope and purpose of all changes

3. **Generate PR description with this structure:**

```markdown
## Summary
Brief overview of what this PR accomplishes (2-3 sentences).

## Changes
- **Feature/Fix Name**: Description of change
- **Another Change**: Description
- List all significant changes

## Technical Details
- Implementation approach or architectural decisions
- Key files modified
- Any dependencies added or updated

## Testing
- [ ] Tested in Extension Development Host
- [ ] Manual testing completed
- [ ] Cross-platform testing (if applicable)
- [ ] No console errors or warnings

## Screenshots (if applicable)
Add screenshots for UI changes

## Breaking Changes
None / List any breaking changes

## Checklist
- [ ] Code follows project style guidelines
- [ ] TypeScript strict mode compliance (no `any` types)
- [ ] Error handling implemented
- [ ] Documentation updated (CLAUDE.md, README.md)
- [ ] No debugging code or console.log statements
- [ ] Tested thoroughly
```

4. **Create the draft PR:**
```bash
gh pr create --draft \
  --title "<conventional-type>: <brief description>" \
  --body "<generated-description>"
```

## PR Title Format

Use conventional commit format:
```
<type>: <description>
```

**Examples:**
- `feat: add visual hook builder with multi-step wizard`
- `fix: resolve file watcher crash on Windows`
- `refactor: extract file discovery logic to service`
- `docs: comprehensive VS Code extension development guide`

## PR Best Practices

### Title
- Use conventional commit type (feat, fix, docs, refactor, etc.)
- Keep under 72 characters
- Be specific and descriptive
- Lowercase except proper nouns

### Description
- **Comprehensive but concise**: Include all relevant information
- **Context**: Why is this change needed?
- **Approach**: How was it implemented?
- **Testing**: What testing was performed?
- **Screenshots**: For any UI changes
- **Breaking changes**: Clearly document if applicable

### Before Creating
- Ensure all tests pass
- Remove debugging code
- Update documentation
- Rebase on latest main if needed
- Review your own changes first

## Output

After creating the draft PR:
1. Display the PR URL
2. Show PR number and title
3. List changed files count
4. Show next steps:
   - Request review when ready
   - Convert from draft to ready for review
   - Address any CI/CD feedback

## Example

```bash
✓ Created draft pull request

PR #42: feat: add visual hook builder
https://github.com/username/claude-code-manager/pull/42

📝 Files changed: 5
   - src/services/hooksService.ts
   - src/providers/claudeTreeDataProvider.ts
   - src/extension.ts
   - CLAUDE.md
   - README.md

Next steps:
1. Verify CI checks pass
2. Self-review the changes on GitHub
3. Mark as "Ready for review" when complete
4. Request review from team members
```

## Notes

- Draft PRs can be worked on without notifying reviewers
- Convert to ready when CI passes and you've self-reviewed
- Link related issues with "Closes #123" in description
- Keep PRs focused - split large changes into multiple PRs