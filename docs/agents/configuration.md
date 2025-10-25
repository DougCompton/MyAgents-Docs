# Agent Configuration Reference

Complete JSON schema reference for agent configuration.

## Schema Overview

```json
{
  "id": "string (required)",
  "name": "string (required)",
  "description": "string (required)",
  "interactive": "boolean (optional)",
  "settings": "array (optional)",
  "agents": "array (required)"
}
```

## Root Properties

### `id` (required)

**Type:** `string`

**Description:** Unique identifier for the agent used in routing and system references.

**Constraints:**
- Must be globally unique
- Lowercase letters, numbers, and hyphens only
- No spaces or special characters
- Descriptive and URL-safe

**Examples:**
```json
✅ "id": "grocery-assistant"
✅ "id": "blog-team"
✅ "id": "customer-service-agent"

❌ "id": "Grocery Assistant"  // Contains spaces and capitals
❌ "id": "ga"                  // Too short/not descriptive
❌ "id": "grocery_assistant"   // Use hyphens, not underscores
```

---

### `name` (required)

**Type:** `string`

**Description:** User-facing display name for the agent.

**Constraints:**
- Clear and professional
- Can contain spaces, capitals, punctuation
- Should immediately convey purpose
- Under 50 characters recommended

**Examples:**
```json
✅ "name": "Grocery Shopping Assistant"
✅ "name": "Business Report Generator"
✅ "name": "Customer Service"

❌ "name": "GA"        // Too short
❌ "name": ""          // Empty not allowed
```

---

### `description` (required)

**Type:** `string`

**Description:** Brief summary of agent capabilities. Used by switchboard for routing decisions.

**Constraints:**
- Single sentence preferred
- Under 150 characters recommended
- Focus on what agent does for users
- Clear and specific

**Examples:**
```json
✅ "description": "Assists customers with grocery shopping, product recommendations, and meal planning"
✅ "description": "Routes customer support tickets to appropriate specialists based on issue type"

❌ "description": "Agent"        // Too vague
❌ "description": ""             // Empty not allowed
```

---

### `interactive` (optional)

**Type:** `boolean`

**Description:** Controls post-processing agent switching behavior.

**Default:** `false` (if omitted)

**Values:**
- `true`: Agent remains active after processing messages
- `false`: Returns to switchboard after processing

**Validation:**
- Must be boolean literal (`true` or `false`)
- Not string (`"true"` or `"false"`)
- Can be omitted (defaults to `false`)

**Examples:**
```json
✅ "interactive": true
✅ "interactive": false
✅ // Property omitted entirely

❌ "interactive": "true"    // String, not boolean
❌ "interactive": 1         // Number, not boolean
❌ "interactive": null      // Null not allowed
```

---

### `settings` (optional)

**Type:** `array` of setting objects

**Description:** Configurable options that modify agent behavior at runtime.

**Schema:**
```json
{
  "name": "string (required)",
  "title": "string (required)",
  "type": "string (required, 'bool' | 'string' | 'number')",
  "description": "string (required)",
  "defaultValue": "any (required, must match type)"
}
```

**Validation Rules:**
1. Each setting name must be unique within agent
2. `defaultValue` must match `type`
3. Only `bool`, `string`, and `number` types supported
4. All fields required for each setting

**Examples:**

Boolean setting:
```json
{
  "name": "enableWebSearch",
  "title": "Enable Web Search",
  "type": "bool",
  "description": "Allow agent to search the web for information",
  "defaultValue": true
}
```

String setting:
```json
{
  "name": "responseFormat",
  "title": "Response Format",
  "type": "string",
  "description": "Format for responses (markdown, plain, html)",
  "defaultValue": "markdown"
}
```

Number setting:
```json
{
  "name": "maxResults",
  "title": "Maximum Results",
  "type": "number",
  "description": "Maximum number of items to return",
  "defaultValue": 10
}
```

**Common Errors:**

```json
❌ {
  "name": "enabled",
  "type": "bool",
  "defaultValue": "true"  // String, should be boolean
}

❌ {
  "name": "count",
  "type": "number",
  "defaultValue": "5"     // String, should be number
}

❌ {
  "name": "setting1",
  "title": "Setting 1",
  "type": "bool"
  // Missing description and defaultValue
}
```

---

### `agents` (required)

**Type:** `array` of persona objects

**Description:** Collection of specialized personas that define agent capabilities.

**Minimum:** 1 persona required

**Schema:** See [Persona Properties](#persona-properties) below

---

## Persona Properties

Each persona in the `agents` array must follow this schema:

```json
{
  "name": "string (required)",
  "description": "string (required)",
  "instructions": "array (required)",
  "model": "string (optional)",
  "subAgents": "array<string> (optional)",
  "toolsets": "array (optional)"
}
```

### `name` (required)

**Type:** `string`

**Description:** Unique identifier for the persona within the agent scope.

**Constraints:**
- Lowercase with underscores (snake_case)
- Descriptive role-based name
- Under 30 characters
- Unique within agent

**Examples:**
```json
✅ "name": "product_specialist"
✅ "name": "data_analyst"
✅ "name": "content_writer"

❌ "name": "Product Specialist"  // Use underscores, not spaces
❌ "name": "ps"                  // Too short
```

---

### `description` (required)

**Type:** `string`

**Description:** Concise summary of persona responsibilities.

**Constraints:**
- Single sentence preferred
- Under 100 characters
- Focus on primary function
- Use active voice

**Examples:**
```json
✅ "description": "Searches and recommends grocery products based on customer needs"
✅ "description": "Analyzes data and generates insights from metrics"

❌ "description": "Does stuff"  // Too vague
```

---

### `instructions` (required)

**Type:** `array` of instruction objects

**Description:** Defines persona behavior, constraints, and guidelines.

**Instruction Schema:**
```json
{
  "content": "string (required)",
  "if": "string (optional, conditional expression)"
}
```

**Basic instruction:**
```json
{
  "content": "You are a helpful assistant. Answer questions clearly and concisely."
}
```

**Conditional instruction:**
```json
{
  "content": "\nUse formal, professional language.",
  "if": "setting.formalMode == true"
}
```

**Conditional Expression Syntax:**
```
setting.<settingName> <operator> <value>
```

**Operators:**
- `==` (equals)
- `!=` (not equals)

**Examples:**
```json
✅ "if": "setting.budgetMode == true"
✅ "if": "setting.dietaryMode == \"vegan\""
✅ "if": "setting.maxResults == 10"

❌ "if": "budgetMode == true"           // Missing "setting." prefix
❌ "if": "setting.mode == vegan"        // String values need quotes
❌ "if": "setting.value = true"         // Single = is assignment
❌ "if": "true"                         // Invalid - just use unconditional
```

**Best Practices:**
1. Start with role definition and scope
2. Specify business rules and constraints
3. Define output format expectations
4. Use conditional instructions for configurability

**Example:**
```json
{
  "instructions": [
    {
      "content": "You are a product specialist. Responsibilities:\n- Search for products\n- Provide 3-5 recommendations\n- Include pricing and availability\n- Format as numbered list"
    },
    {
      "content": "\nBUDGET MODE: Prioritize value and cost-effective options.",
      "if": "setting.budgetMode == true"
    }
  ]
}
```

---

### `model` (optional)

**Type:** `string`

**Description:** Specifies LLM provider for this persona.

**Allowed Values:**
- `"anthropic"` - Claude (Anthropic)
- `"gpt-4o"` - GPT-4 (OpenAI)

**Default:** System default model

**Example:**
```json
{
  "name": "analyst",
  "model": "anthropic",
  "description": "Analyzes data",
  "instructions": [...]
}
```

---

### `subAgents` (optional)

**Type:** `array` of strings

**Description:** Lists personas this persona can delegate to via `transfer_task`.

**Constraints:**
- Must reference valid persona names within same agent
- Names are case-sensitive
- Used by coordinator personas

**Example:**
```json
{
  "name": "coordinator",
  "description": "Manages workflow",
  "subAgents": ["researcher", "writer", "editor"],
  "instructions": [...]
}
```

---

### `toolsets` (optional)

**Type:** `array` of strings or objects

**Description:** External tools available to this persona.

**Simple format (strings):**
```json
{
  "toolsets": ["duckduckgo", "memory-server"]
}
```

**Advanced format (with exclusions):**
```json
{
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
```

**Available toolsets:**
- `duckduckgo` - Web search
- `memory-server` - Persistent storage
- `notification-server` - Push notifications
- `weather-tool` - Weather data
- `google-calendar` - Calendar (requires OAuth)

See [Available Tools](../tools/available-tools.md) for complete list.

---

## Complete Example

```json
{
  "id": "product-assistant",
  "name": "Product Shopping Assistant",
  "description": "Helps customers find and compare products based on their needs",
  "interactive": true,
  "settings": [
    {
      "name": "budgetMode",
      "title": "Budget-Conscious Mode",
      "type": "bool",
      "description": "Prioritize value and cost-effective options",
      "defaultValue": false
    },
    {
      "name": "priceRange",
      "title": "Price Range",
      "type": "string",
      "description": "Preferred price range (budget, moderate, premium)",
      "defaultValue": "moderate"
    }
  ],
  "agents": [
    {
      "name": "shopping_coordinator",
      "description": "Manages product search and recommendations",
      "model": "anthropic",
      "instructions": [
        {
          "content": "You coordinate product shopping. Delegate to product_searcher for finding products. Present results clearly to users."
        }
      ],
      "subAgents": ["product_searcher"]
    },
    {
      "name": "product_searcher",
      "description": "Searches for and evaluates products",
      "model": "anthropic",
      "instructions": [
        {
          "content": "Search for products using web search. Provide 3-5 recommendations with pricing, ratings, and availability."
        },
        {
          "content": "\nPrioritize budget-friendly options and calculate per-unit pricing.",
          "if": "setting.budgetMode == true"
        }
      ],
      "toolsets": ["duckduckgo"]
    }
  ]
}
```

## Validation Checklist

Before deploying your agent:

- [ ] Valid JSON syntax
- [ ] All required fields present
- [ ] Unique `id` across all agents
- [ ] Each persona has `name`, `description`, `instructions`
- [ ] Boolean values are unquoted
- [ ] Setting types match `defaultValue` types
- [ ] Setting names unique within agent
- [ ] Conditional expressions valid
- [ ] Referenced toolsets exist
- [ ] SubAgent names match persona names

## Common Validation Errors

### Missing Required Field
```
Error: "Missing required agent field: description"
```
**Fix:** Add all required properties (`id`, `name`, `description`, `agents`)

### Type Mismatch
```
Error: "interactive property must be a boolean, got: string"
```
**Fix:** Use `true`/`false` not `"true"`/`"false"`

### Setting Type Mismatch
```
Error: "settings[0]: type is 'bool' but defaultValue is string"
```
**Fix:** Match types: `{"type": "bool", "defaultValue": true}`

### Invalid Conditional Expression
```
Error: "Invalid conditional expression in persona 'searcher'"
```
**Fix:** Use correct syntax: `setting.name == value`

### Duplicate Setting Names
```
Error: "Agent.settings contains duplicate setting names: budgetMode"
```
**Fix:** Ensure each setting name is unique

### Unknown Toolset
```
Error: "Unknown or disabled MCP toolset keys: invalid-tool"
```
**Fix:** Verify toolset name and availability

---

**Next:** [Examples](examples.md) | [Settings Guide](../advanced/settings.md) | [Conditional Instructions](../advanced/conditional-instructions.md)

