# Add Tree Data Provider

Generate a new VS Code TreeDataProvider with all necessary boilerplate.

## Instructions

1. **Gather requirements:**
   - Ask for the provider name (e.g., "Workflows", "Templates")
   - Determine what data it will display
   - Identify the data source (files, API, service, etc.)

2. **Update type definitions in `src/core/types.ts`:**
```typescript
export interface <ProviderName>Item {
  name: string;
  path?: string;
  type: 'folder' | 'file' | 'item';
  children?: <ProviderName>Item[];
  // Add other relevant fields
}
```

3. **Add provider class in `src/providers/claudeTreeDataProvider.ts`:**
```typescript
export class <ProviderName>TreeProvider implements vscode.TreeDataProvider<<ProviderName>Item> {
  private _onDidChangeTreeData = new vscode.EventEmitter<<ProviderName>Item | undefined>();
  readonly onDidChangeTreeData = this._onDidChangeTreeData.event;

  constructor(
    // Inject required services
  ) {}

  refresh(): void {
    this._onDidChangeTreeData.fire(undefined);
  }

  getTreeItem(element: <ProviderName>Item): vscode.TreeItem {
    const treeItem = new vscode.TreeItem(
      element.name,
      element.children ? vscode.TreeItemCollapsibleState.Collapsed : vscode.TreeItemCollapsibleState.None
    );

    // Set icon
    treeItem.iconPath = new vscode.ThemeIcon('file'); // Choose appropriate icon

    // Set command for click action
    if (element.path) {
      treeItem.command = {
        command: 'claudeCodeManager.openFile',
        title: 'Open',
        arguments: [element]
      };
    }

    // Set context value for context menu
    treeItem.contextValue = element.type;

    return treeItem;
  }

  async getChildren(element?: <ProviderName>Item): Promise<<ProviderName>Item[]> {
    if (!element) {
      // Return root level items
      return this.getRootItems();
    }

    // Return children of element
    return element.children || [];
  }

  private async getRootItems(): Promise<<ProviderName>Item[]> {
    // Implement data fetching logic
    return [];
  }
}
```

4. **Register view in `package.json`:**
```json
{
  "contributes": {
    "views": {
      "claude-code-manager": [
        {
          "id": "claudeCodeManager.<viewId>",
          "name": "<Display Name>",
          "icon": "resources/icons/icon.svg"
        }
      ]
    }
  }
}
```

5. **Register provider in `src/extension.ts` activate function:**
```typescript
// Create provider instance
const <providerName>Provider = new <ProviderName>TreeProvider(/* dependencies */);

// Register tree view
const <viewId>TreeView = vscode.window.createTreeView('claudeCodeManager.<viewId>', {
  treeDataProvider: <providerName>Provider,
  showCollapseAll: true
});

// Register refresh command
context.subscriptions.push(
  vscode.commands.registerCommand('claudeCodeManager.refresh<ProviderName>', () => {
    <providerName>Provider.refresh();
  })
);

// Add to subscriptions
context.subscriptions.push(<viewId>TreeView);
```

6. **Add refresh button in `package.json`:**
```json
{
  "contributes": {
    "menus": {
      "view/title": [
        {
          "command": "claudeCodeManager.refresh<ProviderName>",
          "when": "view == claudeCodeManager.<viewId>",
          "group": "navigation"
        }
      ]
    }
  }
}
```

## TreeView Icons

Common VS Code built-in icons:
- `file` - Generic file
- `folder` - Folder
- `symbol-class` - Class/component
- `symbol-method` - Function
- `gear` - Settings
- `book` - Documentation
- `tools` - Tools/utilities
- `extensions` - Extensions/plugins
- `database` - Data/storage
- `organization` - Groups/teams

See full list: https://code.visualstudio.com/api/references/icons-in-labels

## Context Values

Set `treeItem.contextValue` to enable context menu items:
```typescript
treeItem.contextValue = 'workflowFile'; // Used in when clauses
```

Then add context menu in `package.json`:
```json
{
  "contributes": {
    "menus": {
      "view/item/context": [
        {
          "command": "claudeCodeManager.editWorkflow",
          "when": "view == claudeCodeManager.workflows && viewItem == workflowFile",
          "group": "inline"
        }
      ]
    }
  }
}
```

## Output

Show summary of created/modified files:
- `src/core/types.ts` - Added interface
- `src/providers/claudeTreeDataProvider.ts` - Added provider class
- `package.json` - Added view and commands
- `src/extension.ts` - Registered provider

## Testing

1. Press F5 to launch Extension Development Host
2. Open the new tree view in sidebar
3. Verify items display correctly
4. Test refresh command
5. Test click actions
6. Test context menu items