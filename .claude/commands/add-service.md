# Add Service

Create a new service class following the project's service layer pattern.

## Instructions

1. **Determine service purpose:**
   - Ask what domain or functionality this service will handle
   - Identify what data it will manage or operations it will perform
   - Determine dependencies (other services, external APIs, file system)

2. **Create service file in `src/services/`:**
   - Use kebab-case naming: `<name>-service.ts` (e.g., `workflow-service.ts`)
   - Follow the established service pattern

3. **Service template:**
```typescript
import * as vscode from 'vscode';
import * as fs from 'fs/promises';
import * as path from 'path';

// Import types
import type { <ServiceDataType> } from '../core/types';

/**
 * <ServiceName>Service handles <description of responsibility>
 *
 * Responsibilities:
 * - <Primary responsibility>
 * - <Secondary responsibility>
 * - <Additional responsibilities>
 */
export class <ServiceName>Service {
  constructor(
    // Inject dependencies if needed
  ) {}

  /**
   * <Method description>
   * @param param1 Description of param1
   * @returns Description of return value
   * @throws Description of potential errors
   */
  async mainMethod(param1: string): Promise<<ReturnType>> {
    try {
      // Implementation
      return result;
    } catch (error) {
      // Provide meaningful error context
      throw new Error(`Failed to <operation>: ${error instanceof Error ? error.message : String(error)}`);
    }
  }

  // Additional methods...

  /**
   * Private helper method
   */
  private async helperMethod(): Promise<void> {
    // Implementation
  }
}
```

4. **Add type definitions to `src/core/types.ts` if needed:**
```typescript
export interface <ServiceDataType> {
  // Define structure
}

export type <ServiceOptionType> = {
  // Define options
};
```

5. **Import and instantiate in `src/extension.ts` if globally needed:**
```typescript
import { <ServiceName>Service } from './services/<name>-service';

// In activate()
const <serviceName>Service = new <ServiceName>Service(/* dependencies */);

// Pass to providers or commands that need it
```

## Service Design Principles

### 1. Single Responsibility
Each service should handle one domain or concern:
- ✅ `FileDiscoveryService` - Discovers and categorizes files
- ✅ `HooksService` - Manages hook configurations
- ❌ `FileAndHooksService` - Too broad, split into separate services

### 2. Async by Default
All I/O operations must be async:
```typescript
// Good
async readConfig(): Promise<Config> {
  const content = await fs.readFile(path, 'utf-8');
  return JSON.parse(content);
}

// Bad
readConfigSync(): Config {
  const content = fs.readFileSync(path, 'utf-8'); // Blocks extension host
  return JSON.parse(content);
}
```

### 3. Error Handling
Provide context in errors:
```typescript
// Good
try {
  await fs.readFile(configPath, 'utf-8');
} catch (error) {
  throw new Error(`Failed to read config at ${configPath}: ${error.message}`);
}

// Bad
try {
  await fs.readFile(configPath, 'utf-8');
} catch (error) {
  throw error; // No context about what failed
}
```

### 4. Dependency Injection
Inject dependencies in constructor:
```typescript
// Good - testable, flexible
constructor(
  private fileSystem: FileSystemService,
  private logger: LoggerService
) {}

// Bad - hard to test, tightly coupled
constructor() {
  this.fileSystem = new FileSystemService(); // Creates dependency internally
}
```

### 5. Type Safety
Use TypeScript types for all parameters and returns:
```typescript
// Good
async fetchItems(): Promise<Item[]> {
  return items;
}

// Bad
async fetchItems(): Promise<any> { // Avoid 'any'
  return items;
}
```

## Common Service Patterns

### File System Service
```typescript
export class FileSystemService {
  async readFile(filePath: string): Promise<string> {
    return await fs.readFile(filePath, 'utf-8');
  }

  async writeFile(filePath: string, content: string): Promise<void> {
    await fs.mkdir(path.dirname(filePath), { recursive: true });
    await fs.writeFile(filePath, content, 'utf-8');
  }

  async exists(filePath: string): Promise<boolean> {
    try {
      await fs.access(filePath);
      return true;
    } catch {
      return false;
    }
  }
}
```

### Configuration Service
```typescript
export class ConfigService {
  async getConfig<T>(key: string): Promise<T | undefined> {
    const config = vscode.workspace.getConfiguration('claudeCodeManager');
    return config.get<T>(key);
  }

  async setConfig<T>(key: string, value: T): Promise<void> {
    const config = vscode.workspace.getConfiguration('claudeCodeManager');
    await config.update(key, value, vscode.ConfigurationTarget.Global);
  }
}
```

### API Service
```typescript
export class ApiService {
  constructor(private baseUrl: string) {}

  async fetchData<T>(endpoint: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    if (!response.ok) {
      throw new Error(`API request failed: ${response.statusText}`);
    }
    return response.json();
  }
}
```

## Testing Services

```typescript
// Example test structure
describe('WorkflowService', () => {
  let service: WorkflowService;
  let mockFileSystem: FileSystemService;

  beforeEach(() => {
    mockFileSystem = createMockFileSystem();
    service = new WorkflowService(mockFileSystem);
  });

  it('should load workflows from directory', async () => {
    // Test implementation
  });
});
```

## Output

After creating the service:
1. Show file path: `src/services/<name>-service.ts`
2. List public methods created
3. Show integration points (where to import/use)
4. Suggest next steps (write tests, integrate with providers)

## Checklist

- [ ] Service class created in `src/services/`
- [ ] Type definitions added to `src/core/types.ts`
- [ ] All methods are async for I/O operations
- [ ] Error handling implemented with meaningful messages
- [ ] JSDoc comments for public methods
- [ ] TypeScript strict mode compliant (no `any` types)
- [ ] Imported and instantiated in appropriate location
- [ ] Tested in Extension Development Host