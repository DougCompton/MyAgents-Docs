# JSON Schema Reference

Complete technical reference for agent configuration schema.

## Table of Contents

- [Root Schema](#root-schema)
- [Setting Schema](#setting-schema)
- [Persona Schema](#persona-schema)
- [Instruction Schema](#instruction-schema)
- [Toolset Schema](#toolset-schema)
- [Type Definitions](#type-definitions)

## Root Schema

### IAgent Team

The top-level agent configuration object.

```typescript
interface IAgentTeam {
  id: string;
  name: string;
  description: string;
  interactive?: boolean;
  settings?: ISetting[];
  agents: IPersona[];
}
```

#### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Unique identifier for agent |
| `name` | string | Yes | Display name |
| `description` | string | Yes | Agent capabilities summary |
| `interactive` | boolean | No | Controls post-processing behavior |
| `settings` | ISetting[] | No | Runtime configuration options |
| `agents` | IPersona[] | Yes | Array of personas (min 1) |

#### Constraints

- `id`: Must be globally unique, lowercase, hyphens only
- `name`: Under 50 characters recommended
- `description`: Under 150 characters recommended
- `agents`: Minimum 1 persona required

#### Example

```json
{
  "id": "example-agent",
  "name": "Example Agent",
  "description": "Demonstrates agent configuration",
  "interactive": true,
  "settings": [],
  "agents": [...]
}
```

---

## Setting Schema

### ISetting

Runtime configuration option for agents.

```typescript
interface ISetting {
  name: string;
  title: string;
  type: 'bool' | 'string' | 'number';
  description: string;
  defaultValue: boolean | string | number;
}
```

#### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Internal identifier (camelCase) |
| `title` | string | Yes | User-facing display name |
| `type` | enum | Yes | One of: 'bool', 'string', 'number' |
| `description` | string | Yes | Detailed explanation |
| `defaultValue` | any | Yes | Must match type |

#### Constraints

- `name`: Must be unique within agent, camelCase
- `type`: Only 'bool', 'string', or 'number' allowed
- `defaultValue`: Must match specified type

#### Type Matching

```typescript
// Boolean setting
{
  type: 'bool',
  defaultValue: true | false  // Must be boolean
}

// String setting
{
  type: 'string',
  defaultValue: "any string"  // Must be string
}

// Number setting
{
  type: 'number',
  defaultValue: 42  // Must be number
}
```

#### Example

```json
{
  "name": "maxResults",
  "title": "Maximum Results",
  "type": "number",
  "description": "Maximum number of items to return",
  "defaultValue": 10
}
```

---

## Persona Schema

### IPersona

Specialized role within an agent.

```typescript
interface IPersona {
  name: string;
  description: string;
  instructions: IInstruction[];
  model?: string;
  subAgents?: string[];
  toolsets?: (string | IToolsetConfig)[];
}
```

#### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Unique identifier (snake_case) |
| `description` | string | Yes | Role description |
| `instructions` | IInstruction[] | Yes | Behavior definitions |
| `model` | string | No | LLM provider |
| `subAgents` | string[] | No | Delegatable personas |
| `toolsets` | array | No | Available tools |

#### Constraints

- `name`: Must be unique within agent, snake_case
- `model`: Must be 'anthropic' or 'gpt-4o'
- `subAgents`: Must reference valid persona names
- `toolsets`: Must reference registered toolsets

#### Example

```json
{
  "name": "coordinator",
  "description": "Manages workflow",
  "model": "anthropic",
  "instructions": [...],
  "subAgents": ["specialist_a", "specialist_b"],
  "toolsets": ["duckduckgo"]
}
```

---

## Instruction Schema

### IInstruction

Defines persona behavior and constraints.

```typescript
interface IInstruction {
  content: string;
  if?: string;
}
```

#### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `content` | string | Yes | Instruction text |
| `if` | string | No | Conditional expression |

#### Conditional Expression Syntax

```
setting.<settingName> <operator> <value>
```

**Operators:**
- `==` : Equality
- `!=` : Inequality

**Examples:**
```typescript
"if": "setting.enabled == true"
"if": "setting.mode == \"advanced\""
"if": "setting.count == 10"
"if": "setting.enabled != false"
```

#### Constraints

- `content`: Non-empty string
- `if`: Must use valid expression syntax
- String values in expressions require escaped quotes

#### Example

```json
{
  "content": "You are a helpful assistant...",
  "if": "setting.verboseMode == true"
}
```

---

## Toolset Schema

### Simple Toolset

String reference to registered toolset:

```typescript
type SimpleToolset = string;
```

```json
"toolsets": ["duckduckgo", "memory-server"]
```

### Advanced Toolset

Toolset with conditional exclusions:

```typescript
interface IToolsetConfig {
  name: string;
  exclude?: IToolExclusion[];
}

interface IToolExclusion {
  tool: string;
  if: string;
}
```

#### Example

```json
{
  "name": "memory-server",
  "exclude": [
    {
      "tool": "delete",
      "if": "setting.readOnlyMode == true"
    }
  ]
}
```

---

## Type Definitions

### Supported Types

```typescript
// Setting types
type SettingType = 'bool' | 'string' | 'number';

// Setting values
type SettingValue = boolean | string | number;

// LLM models
type Model = 'anthropic' | 'gpt-4o';

// Conditional operators
type ConditionalOperator = '==' | '!=';
```

### Validation Rules

#### Type Consistency

```typescript
// Valid
{
  type: 'bool',
  defaultValue: true
}

// Invalid - type mismatch
{
  type: 'bool',
  defaultValue: "true"  // ❌ String, not boolean
}
```

#### Unique Names

```typescript
// Valid
{
  settings: [
    { name: 'setting1', ... },
    { name: 'setting2', ... }
  ]
}

// Invalid - duplicate names
{
  settings: [
    { name: 'setting1', ... },
    { name: 'setting1', ... }  // ❌ Duplicate
  ]
}
```

#### Required Fields

```typescript
// Valid
{
  name: 'example',
  title: 'Example',
  type: 'bool',
  description: 'An example setting',
  defaultValue: false
}

// Invalid - missing fields
{
  name: 'example',
  type: 'bool'
  // ❌ Missing title, description, defaultValue
}
```

---

## Complete Example

```json
{
  "id": "complete-example",
  "name": "Complete Example Agent",
  "description": "Demonstrates all schema features",
  "interactive": true,
  "settings": [
    {
      "name": "enableFeature",
      "title": "Enable Feature",
      "type": "bool",
      "description": "Toggle feature on/off",
      "defaultValue": true
    },
    {
      "name": "operationMode",
      "title": "Operation Mode",
      "type": "string",
      "description": "Mode of operation (basic, advanced, expert)",
      "defaultValue": "basic"
    },
    {
      "name": "maxItems",
      "title": "Maximum Items",
      "type": "number",
      "description": "Maximum number of items to process",
      "defaultValue": 10
    }
  ],
  "agents": [
    {
      "name": "coordinator",
      "description": "Coordinates workflow",
      "model": "anthropic",
      "instructions": [
        {
          "content": "You coordinate tasks. Base behavior..."
        },
        {
          "content": "\nFeature enabled: Additional behavior...",
          "if": "setting.enableFeature == true"
        },
        {
          "content": "\nAdvanced mode: Expert behavior...",
          "if": "setting.operationMode == \"advanced\""
        }
      ],
      "subAgents": ["specialist"],
      "toolsets": []
    },
    {
      "name": "specialist",
      "description": "Handles specialized tasks",
      "model": "anthropic",
      "instructions": [
        {
          "content": "You handle specialized tasks..."
        }
      ],
      "toolsets": [
        "duckduckgo",
        {
          "name": "memory-server",
          "exclude": [
            {
              "tool": "delete",
              "if": "setting.readOnlyMode == true"
            }
          ]
        }
      ]
    }
  ]
}
```

---

## Validation

### Runtime Validation

The system validates:

1. **Required fields** - All mandatory properties present
2. **Type consistency** - Values match declared types
3. **Unique names** - No duplicate setting/persona names
4. **Reference validity** - SubAgents and toolsets exist
5. **Expression syntax** - Conditional expressions valid
6. **Schema compliance** - Follows JSON schema

### Validation Errors

Common validation errors and resolutions:

**Missing required field:**
```
Error: "Missing required agent field: description"
Solution: Add all required properties
```

**Type mismatch:**
```
Error: "interactive property must be a boolean, got: string"
Solution: Use true/false, not "true"/"false"
```

**Duplicate name:**
```
Error: "Agent.settings contains duplicate setting names: mode"
Solution: Ensure unique names
```

**Invalid expression:**
```
Error: "Invalid conditional expression: mode == true"
Solution: Use "setting.mode == true"
```

---

## Next Steps

- **[API Reference](api.md)** - REST API endpoints
- **[Agent Configuration](../agents/configuration.md)** - Practical guide
- **[Creating Agents](../agents/creating-agents.md)** - Step-by-step guide

---

**Related:**  
[Configuration Guide](../agents/configuration.md) | [API Reference](api.md) | [Agent Examples](../agents/examples.md)

