# Test Extension

Launch the Extension Development Host to test the VS Code extension in a clean environment.

## Instructions

1. **Ensure clean build state:**
```bash
# Clean and rebuild
rm -rf dist/
npm run build
```

2. **Launch Extension Development Host:**
   - **Option A**: Press `F5` in VS Code (recommended)
   - **Option B**: Run from Command Palette:
     1. `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
     2. Type "Debug: Start Debugging"
     3. Select "Run Extension"

3. **What happens:**
   - A new VS Code window opens (Extension Development Host)
   - The extension is loaded in this window
   - Original VS Code window shows Debug Console with extension logs
   - Breakpoints in TypeScript code are active

4. **Testing checklist:**

   **Basic Functionality:**
   - [ ] Extension activates (check Debug Console for "Extension activated")
   - [ ] Activity bar icon appears (Claude logo)
   - [ ] Tree views render in sidebar
   - [ ] No errors in Debug Console

   **Memories Tree:**
   - [ ] Global CLAUDE.md files displayed
   - [ ] Project CLAUDE.md files displayed
   - [ ] Markdown sections expandable
   - [ ] Click to open file works
   - [ ] Refresh button works

   **Commands Tree:**
   - [ ] Global commands displayed
   - [ ] Project commands displayed
   - [ ] Folders expand/collapse
   - [ ] Create new command works
   - [ ] Open command file works

   **Skills Tree:**
   - [ ] Skills folders displayed
   - [ ] SKILL.md files detected
   - [ ] YAML frontmatter parsed
   - [ ] Description tooltips show
   - [ ] Create skill works

   **Sub-Agents Tree:**
   - [ ] Agents displayed with colors
   - [ ] Color-coded icons render
   - [ ] Create agent works
   - [ ] Folders supported

   **Hooks Tree:**
   - [ ] Hooks organized by location/event
   - [ ] Create hook wizard works
   - [ ] Edit hook works
   - [ ] Delete hook works
   - [ ] Duplicate hook works

   **Permissions Tree:**
   - [ ] Permissions parsed from settings
   - [ ] Organized by type (allow/ask/deny)
   - [ ] Click to jump to settings works

   **File Operations:**
   - [ ] Create file with template works
   - [ ] Rename file works
   - [ ] Delete file works (with confirmation)
   - [ ] Move between global/project works
   - [ ] Copy file path works
   - [ ] Reveal in Finder/Explorer works
   - [ ] Create folder works

5. **Testing file watcher:**
   - Manually edit a `.claude/` file in terminal
   - Verify tree view auto-refreshes
   - Create new file, confirm it appears
   - Delete file, confirm it disappears

6. **Testing cross-platform paths:**
   - Verify paths work correctly
   - Check file operations don't break
   - Test home directory resolution

7. **Testing error handling:**
   - Try to open non-existent file
   - Try to create file in invalid location
   - Try to parse invalid YAML frontmatter
   - Verify error messages are user-friendly

## Debugging

### Set Breakpoints
- Click in gutter next to line numbers in VS Code
- Red dots indicate active breakpoints
- Code execution pauses at breakpoints
- Inspect variables in Debug sidebar

### Debug Console
Watch for:
- Extension activation messages
- Error stack traces
- Console.log output (remove before production)
- TypeScript compilation errors

### Common Issues

**Extension doesn't activate:**
```
Check Debug Console for:
- "Extension activation failed"
- Missing dependencies
- TypeScript errors
Solution: Fix errors and press Cmd+Shift+F5 to restart
```

**Tree views empty:**
```
Check Debug Console for:
- File discovery errors
- Path resolution issues
Solution: Add console.log in getChildren() to debug
```

**Commands not registered:**
```
Verify:
- Commands in package.json contributes.commands
- Commands registered in activate()
- Command IDs match exactly
Solution: Fix registration and restart
```

**File watcher not working:**
```
Check:
- Workspace folder is open (required for watcher)
- .claude directory exists
- Watcher glob pattern is correct
Solution: Ensure workspace is open, check watcher setup
```

## Development Workflow

1. **Make changes to code**
2. **Restart Extension Development Host:**
   - Press `Cmd+Shift+F5` (Mac) or `Ctrl+Shift+F5` (Windows/Linux)
   - Or click restart button in Debug toolbar
3. **Test changes**
4. **Repeat**

**Hot reload:** The extension does NOT hot reload. You must restart the Extension Development Host after every code change.

## Watch Mode (Recommended)

Run watch mode in a separate terminal:
```bash
npm run watch
```

This will:
- Continuously watch for TypeScript changes
- Automatically rebuild on save
- Faster development cycle

Then just press `Cmd+Shift+F5` to restart and load new build.

## Manual Testing Scenarios

### Scenario 1: Create a New Skill
1. Click `+` in Skills tree view
2. Enter skill name: "test-skill"
3. Verify folder created: `.claude/skills/test-skill/`
4. Verify SKILL.md created with template
5. Edit YAML frontmatter, add description
6. Refresh skills tree
7. Verify description shows in tooltip

### Scenario 2: Create a Hook
1. Click `+` in Hooks tree view
2. Select event type: "PreToolUse"
3. Select tool: "Edit"
4. Enter pattern: "src/**/*.ts"
5. Choose hook type: "Prompt"
6. Enter prompt: "Ensure TypeScript strict mode"
7. Verify hook appears in tree
8. Open settings.local.json, verify JSON structure

### Scenario 3: Move Command to Global
1. Right-click a project command
2. Select "Move to Global"
3. Verify file moved to ~/.claude/commands/
4. Verify file removed from .claude/commands/
5. Refresh commands tree
6. Verify command now under Global section

## Performance Testing

- Test with large number of files (100+ commands)
- Test with deeply nested folder structures
- Test rapid file changes (file watcher)
- Monitor memory usage in Task Manager/Activity Monitor
- Check for memory leaks (reload multiple times)

## Output

After testing, document:
```
✓ Extension activated successfully
✓ All tree views render correctly
✓ File operations work as expected
✓ File watcher responds to changes
✓ Error handling provides clear messages
✓ Cross-platform paths work correctly
✗ Found issue: [Describe issue]

Performance:
- Activation time: ~200ms
- Tree view render: ~50ms
- File operations: <100ms

Tested on:
- macOS 14.0
- VS Code 1.85.0
- Node.js 20.10.0
```

## Checklist

- [ ] Extension activates without errors
- [ ] All tree views display correctly
- [ ] File creation works
- [ ] File editing opens correct file
- [ ] File deletion works (with confirmation)
- [ ] File moving between scopes works
- [ ] File watcher triggers refresh
- [ ] Hooks can be created/edited/deleted
- [ ] Color-coded agents display properly
- [ ] YAML frontmatter parsing works
- [ ] Error messages are user-friendly
- [ ] No console.log statements left in code
- [ ] Performance is acceptable
- [ ] Memory usage is reasonable

## Next Steps

After successful testing:
1. Remove any debugging code
2. Commit changes with /commit
3. Create PR with /draft-pr
4. Package extension with /package-extension (for distribution)

If issues found:
1. Fix issues
2. Restart Extension Development Host
3. Re-test
4. Repeat until all issues resolved