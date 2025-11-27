# Coding Standards

This document outlines the coding standards and conventions used in this project.

## Overview

This project follows the [n8n community node guidelines](https://docs.n8n.io/integrations/creating-nodes/guidelines/) and uses automated tooling to enforce code quality and consistency.

## Language & Framework

- **Language**: TypeScript 5.5.4
- **Target**: ES2019
- **Module System**: CommonJS
- **Framework**: n8n workflow automation

## Code Quality Tools

### TypeScript Configuration

We use strict TypeScript settings to ensure type safety:

```json
{
  "strict": true,
  "noImplicitAny": true,
  "noImplicitReturns": true,
  "noUnusedLocals": true,
  "strictNullChecks": true,
  "forceConsistentCasingInFileNames": true
}
```

**Key rules:**
- All types must be explicitly declared
- No unused variables or imports
- Strict null checks enabled
- Consistent file naming (case-sensitive)

### ESLint

We use ESLint with the n8n community plugin for code quality and n8n-specific rules.

**Plugins:**
- `@typescript-eslint/parser` - TypeScript support
- `eslint-plugin-n8n-nodes-base` - n8n community standards

**Key configurations:**
- **Package.json**: Must follow n8n community package naming conventions
- **Credentials**: Follow n8n credential class standards
- **Nodes**: Follow n8n node development standards

**Run linting:**
```bash
pnpm lint        # Check for issues
pnpm lintfix     # Auto-fix issues where possible
```

### Prettier

We use Prettier for consistent code formatting.

**Configuration:**
```javascript
{
  semi: true,              // Always use semicolons
  trailingComma: 'all',    // Trailing commas where valid (ES5+)
  bracketSpacing: true,    // Spaces in object literals
  useTabs: true,           // Use tabs for indentation
  singleQuote: true,       // Use single quotes
  printWidth: 100,         // Line length limit
  endOfLine: 'lf'          // Unix-style line endings
}
```

**Run formatting:**
```bash
pnpm format      # Format all files
```

## File Structure

```
.
├── credentials/     # n8n credential classes
├── nodes/          # n8n node implementations
├── dist/           # Compiled output (gitignored)
└── node_modules/   # Dependencies (gitignored)
```

## Git Workflow

### Branch Naming
- `main` - Production-ready code
- `feature/[name]` - New features
- `fix/[name]` - Bug fixes
- `chore/[name]` - Maintenance tasks

### Commit Messages
Follow conventional commits:
```
type(scope): description

Examples:
- feat(confluence): add page creation endpoint
- fix(auth): resolve token refresh issue
- chore: update dependencies
- docs: improve setup instructions
```

### Pull Requests
- All PRs must pass CI checks:
  - ✅ Build succeeds
  - ✅ ESLint passes
  - ✅ CodeQL security scan passes
  - ✅ Dependency audit passes

## Development Workflow

### Initial Setup
```bash
pnpm install
```

### Before Committing
```bash
pnpm lint        # Check code quality
pnpm format      # Format code
pnpm build       # Ensure build works
```

### Pre-publish Checks
```bash
pnpm build
pnpm lint -c .eslintrc.prepublish.js nodes credentials package.json
```

## Naming Conventions

### General Rules

#### TypeScript/JavaScript
- **Classes**: PascalCase - `AtlassianCredentialsApi`, `Confluence`
- **Interfaces**: PascalCase with `I` prefix - `ICredentialType`, `INodeType`, `IDataObject`
- **Types**: PascalCase - `DeclarativeRestApiSettings`
- **Functions**: camelCase - `handlePagination`, `processRequest`
- **Variables**: camelCase - `aggregatedResult`, `displayName`
- **Constants**: camelCase or UPPER_SNAKE_CASE - `defaultLimit`, `MAX_RETRIES`

#### Files and Directories
- **Credential files**: `[Name].credentials.ts` - e.g., `AtlassianCredentialsApi.credentials.ts`
- **Node files**: `[Name].node.ts` - e.g., `Confluence.node.ts`
- **Node metadata**: `[Name].node.json` - e.g., `Confluence.node.json`
- **Icons**: lowercase with extension - `confluence.svg`, `atlassian.svg`
- **Directories**: lowercase - `credentials/`, `nodes/`, `dist/`

### n8n-Specific Naming

#### Credential Properties
```typescript
export class AtlassianCredentialsApi implements ICredentialType {
  name = 'atlassianCredentialsApi';        // camelCase, no spaces
  displayName = 'Atlassian Credentials API'; // Human-readable, with spaces
  icon = 'file:atlassian.svg';             // Reference to icon file
}
```

#### Node Properties
```typescript
export class Confluence implements INodeType {
  description: INodeTypeDescription = {
    name: 'confluence',                    // lowercase, no spaces (technical)
    displayName: 'Confluence',             // PascalCase (user-facing)
    group: ['transform'],                  // lowercase category
    subtitle: '={{$parameter["resource"]}}' // Expression format
  }
}
```

#### Parameter Names
- **Technical names** (property `name`): camelCase, no spaces
  - `domain`, `apiToken`, `returnAll`, `pageId`
- **Display names** (property `displayName`): Human-readable with spaces
  - `Domain`, `API Token`, `Return All`, `Page ID`

#### Resource and Operation Names
```typescript
{
  displayName: 'Resource',
  name: 'resource',
  type: 'options',
  options: [
    {
      name: 'Page',      // Singular, PascalCase
      value: 'page',     // lowercase
    }
  ]
}
```

### Package Naming
- **Package name**: Must start with `n8n-nodes-` for community nodes
  - Example: `@bitovi/n8n-nodes-confluence`
- **Scope**: Organization or personal scope (optional)
  - `@username/n8n-nodes-[name]`
  - `@organization/n8n-nodes-[name]`

### Git Conventions
- **Branches**: lowercase with hyphens
  - `main`, `feature/add-search`, `fix/auth-token`, `chore/update-deps`
- **Tags**: Semantic versioning
  - `v0.1.6`, `v1.0.0`

## n8n-Specific Guidelines

### Credentials
- Must extend `ICredentialType`
- Class name must match filename (minus `.credentials.ts`)
- Include clear display names for all fields
- Document all fields with descriptions
- Use appropriate field types:
  - `string` - Text input
  - `password` - Hidden input
  - `options` - Dropdown selection
  - `boolean` - Checkbox
- Property naming:
  ```typescript
  properties: INodeProperties[] = [
    {
      displayName: 'API Token',  // User-facing name
      name: 'apiToken',          // Technical property name
      type: 'password',
      default: '',
    }
  ]
  ```

### Nodes
- Must extend `INodeType`
- Class name must match filename (minus `.node.ts`)
- Follow n8n's property naming conventions (see above)
- Implement proper error handling
- Use `continueOnFail` for resilient workflows
- Document all parameters clearly
- Node structure:
  ```typescript
  export class NodeName implements INodeType {
    description: INodeTypeDescription = {
      displayName: 'Node Name',  // User-facing
      name: 'nodeName',          // Technical (lowercase)
      group: ['transform'],      // Category
      version: 1,                // Node version
      // ... rest of configuration
    }
  }
  ```

### Icon Files
- Must be SVG format
- Named in lowercase: `confluence.svg`, `atlassian.svg`
- Referenced in code as: `icon: 'file:confluence.svg'`
- Stored alongside node/credential files

### Testing
- Test credentials with real API endpoints
- Verify all node operations work correctly
- Test error scenarios
- Validate input parameters
- Test with different parameter combinations

## Code Review Checklist

Before submitting a PR, ensure:

- [ ] Code follows TypeScript strict mode
- [ ] All ESLint rules pass
- [ ] Code is formatted with Prettier
- [ ] Build completes without errors
- [ ] No security vulnerabilities introduced
- [ ] Documentation is updated if needed
- [ ] Commit messages follow conventions

## Security

- Never commit credentials or API keys
- Use environment variables for sensitive data
- Follow the security policy in [SECURITY.md](SECURITY.md)
- Report vulnerabilities privately

## Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community Nodes Guidelines](https://docs.n8n.io/integrations/creating-nodes/guidelines/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Prettier Documentation](https://prettier.io/docs/en/)
- [ESLint Documentation](https://eslint.org/docs/latest/)

## Questions?

If you have questions about these standards, please:
1. Check the existing codebase for examples
2. Review the linked documentation
3. Ask in pull request comments
4. Contact the maintainers

---

*Last updated: November 2025*
