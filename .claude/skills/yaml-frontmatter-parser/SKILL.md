---
name: YAML Frontmatter Parser
description: Parse and validate YAML frontmatter in markdown files for skills and agents
version: 1.0.0
---

# YAML Frontmatter Parser Skill

This skill provides expertise in parsing and validating YAML frontmatter from markdown files, specifically for Claude Code configuration files like skills and sub-agents.

## What is YAML Frontmatter?

YAML frontmatter is metadata at the beginning of markdown files, enclosed in triple dashes:

```markdown
---
name: my-skill
description: A useful skill
version: 1.0.0
author: Developer Name
---

# Skill Content

The rest of the markdown content...
```

## Parsing YAML Frontmatter

### Basic Parser Implementation

```typescript
interface YAMLFrontmatter {
  [key: string]: string | number | boolean | string[];
}

/**
 * Extract YAML frontmatter from markdown content
 */
function extractFrontmatter(content: string): YAMLFrontmatter | null {
  // Match YAML frontmatter pattern
  const frontmatterRegex = /^---\s*\n([\s\S]*?)\n---\s*\n/;
  const match = content.match(frontmatterRegex);

  if (!match || !match[1]) {
    return null;
  }

  const yamlContent = match[1];
  return parseYAML(yamlContent);
}

/**
 * Simple YAML parser for key-value pairs
 * Note: This handles simple cases. For complex YAML, use a library like js-yaml
 */
function parseYAML(yaml: string): YAMLFrontmatter {
  const result: YAMLFrontmatter = {};
  const lines = yaml.split('\n');

  for (const line of lines) {
    // Skip empty lines and comments
    if (!line.trim() || line.trim().startsWith('#')) {
      continue;
    }

    // Parse key-value pair
    const colonIndex = line.indexOf(':');
    if (colonIndex === -1) {
      continue;
    }

    const key = line.substring(0, colonIndex).trim();
    let value: string | number | boolean = line.substring(colonIndex + 1).trim();

    // Remove quotes
    if ((value.startsWith('"') && value.endsWith('"')) ||
        (value.startsWith("'") && value.endsWith("'"))) {
      value = value.slice(1, -1);
    }

    // Parse booleans
    if (value === 'true') value = true;
    if (value === 'false') value = false;

    // Parse numbers
    if (!isNaN(Number(value)) && value !== '') {
      value = Number(value);
    }

    result[key] = value;
  }

  return result;
}
```

### Usage Example

```typescript
import * as fs from 'fs/promises';

// Read markdown file
const content = await fs.readFile('skill.md', 'utf-8');

// Extract frontmatter
const frontmatter = extractFrontmatter(content);

if (frontmatter) {
  console.log('Name:', frontmatter.name);
  console.log('Description:', frontmatter.description);
  console.log('Version:', frontmatter.version);
}

// Remove frontmatter to get body content
const body = content.replace(/^---\s*\n[\s\S]*?\n---\s*\n/, '');
```

## Claude Code Configuration Schemas

### Skill YAML Frontmatter

```yaml
---
name: skill-name                          # Required: Skill identifier
description: Brief description            # Required: One-line description
version: 1.0.0                           # Optional: Semantic version
author: Author Name                       # Optional: Author information
tags: [tag1, tag2]                       # Optional: Categories/tags
dependencies: [other-skill]              # Optional: Required skills
---
```

**Validation:**
```typescript
interface SkillFrontmatter {
  name: string;
  description: string;
  version?: string;
  author?: string;
  tags?: string[];
  dependencies?: string[];
}

function validateSkillFrontmatter(fm: YAMLFrontmatter): SkillFrontmatter {
  if (!fm.name || typeof fm.name !== 'string') {
    throw new Error('Skill frontmatter must include "name" field');
  }

  if (!fm.description || typeof fm.description !== 'string') {
    throw new Error('Skill frontmatter must include "description" field');
  }

  return {
    name: fm.name,
    description: fm.description,
    version: fm.version as string | undefined,
    author: fm.author as string | undefined,
    tags: Array.isArray(fm.tags) ? fm.tags : undefined,
    dependencies: Array.isArray(fm.dependencies) ? fm.dependencies : undefined,
  };
}
```

### Sub-Agent YAML Frontmatter

```yaml
---
name: agent-name                          # Required: Agent identifier
color: blue                               # Optional: Icon color (red, blue, green, purple, yellow, orange)
description: Brief description            # Optional: Agent purpose
model: sonnet                            # Optional: Preferred model
tools: [Read, Write, Bash]               # Optional: Tool restrictions
---
```

**Validation:**
```typescript
interface AgentFrontmatter {
  name: string;
  color?: 'red' | 'blue' | 'green' | 'purple' | 'yellow' | 'orange' | 'default';
  description?: string;
  model?: 'sonnet' | 'opus' | 'haiku';
  tools?: string[];
}

function validateAgentFrontmatter(fm: YAMLFrontmatter): AgentFrontmatter {
  if (!fm.name || typeof fm.name !== 'string') {
    throw new Error('Agent frontmatter must include "name" field');
  }

  // Validate color
  const validColors = ['red', 'blue', 'green', 'purple', 'yellow', 'orange', 'default'];
  if (fm.color && !validColors.includes(fm.color as string)) {
    throw new Error(`Invalid color: ${fm.color}. Must be one of: ${validColors.join(', ')}`);
  }

  return {
    name: fm.name,
    color: (fm.color as AgentFrontmatter['color']) || 'default',
    description: fm.description as string | undefined,
    model: fm.model as AgentFrontmatter['model'] | undefined,
    tools: Array.isArray(fm.tools) ? fm.tools : undefined,
  };
}
```

## File Operations with Frontmatter

### Reading with Frontmatter

```typescript
interface MarkdownFile {
  frontmatter: YAMLFrontmatter | null;
  body: string;
}

async function readMarkdownWithFrontmatter(filePath: string): Promise<MarkdownFile> {
  const content = await fs.readFile(filePath, 'utf-8');
  const frontmatter = extractFrontmatter(content);
  const body = content.replace(/^---\s*\n[\s\S]*?\n---\s*\n/, '');

  return { frontmatter, body };
}
```

### Writing with Frontmatter

```typescript
function serializeYAML(data: YAMLFrontmatter): string {
  const lines: string[] = [];

  for (const [key, value] of Object.entries(data)) {
    if (Array.isArray(value)) {
      lines.push(`${key}: [${value.join(', ')}]`);
    } else if (typeof value === 'string') {
      // Quote strings with special characters
      const needsQuotes = value.includes(':') || value.includes('#') || value.includes('\n');
      lines.push(`${key}: ${needsQuotes ? `"${value}"` : value}`);
    } else {
      lines.push(`${key}: ${value}`);
    }
  }

  return lines.join('\n');
}

async function writeMarkdownWithFrontmatter(
  filePath: string,
  frontmatter: YAMLFrontmatter,
  body: string
): Promise<void> {
  const yaml = serializeYAML(frontmatter);
  const content = `---\n${yaml}\n---\n\n${body}`;
  await fs.writeFile(filePath, content, 'utf-8');
}
```

### Updating Frontmatter

```typescript
async function updateFrontmatter(
  filePath: string,
  updates: Partial<YAMLFrontmatter>
): Promise<void> {
  const { frontmatter, body } = await readMarkdownWithFrontmatter(filePath);

  const updatedFrontmatter = {
    ...(frontmatter || {}),
    ...updates,
  };

  await writeMarkdownWithFrontmatter(filePath, updatedFrontmatter, body);
}

// Usage
await updateFrontmatter('agent.md', { color: 'blue', version: '1.1.0' });
```

## Error Handling

```typescript
function safeExtractFrontmatter(content: string): YAMLFrontmatter | null {
  try {
    const fm = extractFrontmatter(content);
    if (!fm) {
      console.warn('No frontmatter found in content');
      return null;
    }
    return fm;
  } catch (error) {
    console.error('Failed to parse frontmatter:', error);
    return null;
  }
}

async function safeReadSkill(filePath: string): Promise<SkillFrontmatter | null> {
  try {
    const { frontmatter } = await readMarkdownWithFrontmatter(filePath);
    if (!frontmatter) {
      return null;
    }
    return validateSkillFrontmatter(frontmatter);
  } catch (error) {
    console.error(`Failed to read skill at ${filePath}:`, error);
    return null;
  }
}
```

## VS Code Integration

### Display in Tree View

```typescript
// In TreeDataProvider
async getChildren(): Promise<SkillTreeItem[]> {
  const skillFiles = await this.discoverSkillFiles();
  const items: SkillTreeItem[] = [];

  for (const file of skillFiles) {
    try {
      const { frontmatter } = await readMarkdownWithFrontmatter(file.path);

      items.push({
        label: frontmatter?.name || file.name,
        description: frontmatter?.description,
        tooltip: this.createTooltip(frontmatter),
        path: file.path,
      });
    } catch (error) {
      // Handle parsing error gracefully
      items.push({
        label: file.name,
        description: 'Error parsing frontmatter',
        path: file.path,
      });
    }
  }

  return items;
}

private createTooltip(fm: YAMLFrontmatter | null): string {
  if (!fm) return 'No metadata';

  const parts: string[] = [];
  if (fm.name) parts.push(`**${fm.name}**`);
  if (fm.description) parts.push(fm.description);
  if (fm.version) parts.push(`Version: ${fm.version}`);
  if (fm.author) parts.push(`Author: ${fm.author}`);

  return parts.join('\n\n');
}
```

### Color-Coded Agent Icons

```typescript
function getAgentIconColor(color: string): vscode.ThemeColor {
  const colorMap: Record<string, string> = {
    red: 'charts.red',
    blue: 'charts.blue',
    green: 'charts.green',
    purple: 'charts.purple',
    yellow: 'charts.yellow',
    orange: 'charts.orange',
    default: 'foreground',
  };

  return new vscode.ThemeColor(colorMap[color] || colorMap.default);
}

// In getTreeItem
const { frontmatter } = await readMarkdownWithFrontmatter(element.path);
const color = frontmatter?.color || 'default';

treeItem.iconPath = new vscode.ThemeIcon('account', getAgentIconColor(color));
```

## Advanced: Complex YAML

For complex YAML (nested objects, multi-line strings), use a proper YAML library:

```typescript
import * as yaml from 'js-yaml';

function extractFrontmatterAdvanced(content: string): YAMLFrontmatter | null {
  const frontmatterRegex = /^---\s*\n([\s\S]*?)\n---\s*\n/;
  const match = content.match(frontmatterRegex);

  if (!match || !match[1]) {
    return null;
  }

  try {
    return yaml.load(match[1]) as YAMLFrontmatter;
  } catch (error) {
    console.error('YAML parsing error:', error);
    return null;
  }
}
```

## Best Practices

1. **Always validate frontmatter** - Don't assume structure is correct
2. **Provide defaults** - Handle missing optional fields gracefully
3. **Preserve formatting** - When updating, maintain YAML style
4. **Handle errors gracefully** - Show files even if frontmatter is invalid
5. **Use TypeScript types** - Define interfaces for expected structure
6. **Cache parsed frontmatter** - Don't re-parse on every tree refresh
7. **Show helpful errors** - Tell users exactly what's wrong with their YAML

## Usage

This skill provides:
- Frontmatter extraction from markdown
- YAML parsing for key-value pairs
- Validation for skill and agent metadata
- Integration with VS Code tree views
- Color-coded agent icon support
- Safe error handling and defaults