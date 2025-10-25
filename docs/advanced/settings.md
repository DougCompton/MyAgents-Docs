# Settings & Configuration

Master runtime configuration with agent settings.

## What Are Settings?

**Settings** are configurable options that modify agent behavior at runtime without requiring code changes. They enable:

- **Feature toggles** - Enable/disable functionality
- **Behavior customization** - Adjust how agents respond
- **User segmentation** - Different experiences for different users
- **A/B testing** - Compare different configurations

## Setting Types

### Boolean Settings

On/off toggles for features and modes.

```json
{
  "name": "enableWebSearch",
  "title": "Enable Web Search",
  "type": "bool",
  "description": "Allow agent to search the web for information",
  "defaultValue": true
}
```

**Common uses:**
- Feature flags (`enableFeature`, `allowDelete`)
- Mode switches (`strictMode`, `budgetMode`)
- Permissions (`canEdit`, `readOnly`)
- Preferences (`organicPreference`, `saveHistory`)

### String Settings

Text-based configuration for modes and formats.

```json
{
  "name": "outputFormat",
  "title": "Output Format",
  "type": "string",
  "description": "Format for responses (markdown, plain, html)",
  "defaultValue": "markdown"
}
```

**Common uses:**
- Enumerations (`"low"`, `"medium"`, `"high"`)
- Formats (`"markdown"`, `"json"`, `"xml"`)
- Modes (`"casual"`, `"professional"`, `"technical"`)
- Categories (`"vegetarian"`, `"vegan"`, `"keto"`)

**Best practices:**
- Document valid values in description
- Use lowercase for consistency
- Choose clear, descriptive values
- Consider future extensibility

### Number Settings

Numeric configuration for thresholds and limits.

```json
{
  "name": "maxResults",
  "title": "Maximum Results",
  "type": "number",
  "description": "Maximum number of items to return",
  "defaultValue": 10
}
```

**Common uses:**
- Result limits (`maxResults`, `pageSize`)
- Thresholds (`minRating`, `maxPrice`)
- Durations (`timeoutSeconds`, `cacheDuration`)
- Quantities (`batchSize`, `retryCount`)

## Setting Design

### Schema Structure

Each setting requires five properties:

```json
{
  "name": "settingIdentifier",      // Internal key (camelCase)
  "title": "User-Facing Name",      // Display name
  "type": "bool",                   // Type: bool, string, or number
  "description": "Detailed explanation of what this controls",
  "defaultValue": true              // Must match type
}
```

### Naming Conventions

**Setting names (internal):**
- Use camelCase: `enableWebSearch`, `maxResults`
- Descriptive and clear
- No abbreviations unless universally understood
- Consistent with domain terminology

**Titles (user-facing):**
- Use Title Case: `"Enable Web Search"`, `"Maximum Results"`
- Clear and professional
- Brief but informative

**Descriptions:**
- Explain what the setting controls
- Document valid values for strings
- Include units for numbers (seconds, dollars, items)
- Provide context and examples

### Default Values

Choose defaults that work for most users:

```json
{
  "name": "enableNotifications",
  "defaultValue": true    // Most users want notifications
}
```

```json
{
  "name": "debugMode",
  "defaultValue": false   // Debug should be opt-in
}
```

```json
{
  "name": "resultsPerPage",
  "defaultValue": 20      // Balanced between performance and info
}
```

## Using Settings in Instructions

Settings become available to conditional instructions via the `setting.` prefix.

### Basic Conditional

```json
{
  "instructions": [
    {
      "content": "Base instructions that always apply..."
    },
    {
      "content": "\nAdditional behavior when feature is enabled...",
      "if": "setting.enableFeature == true"
    }
  ]
}
```

### Multiple Conditions

```json
{
  "settings": [
    {
      "name": "mode",
      "title": "Operation Mode",
      "type": "string",
      "description": "Mode of operation (basic, advanced, expert)",
      "defaultValue": "basic"
    }
  ],
  "agents": [
    {
      "name": "assistant",
      "instructions": [
        {
          "content": "Base instructions..."
        },
        {
          "content": "\nBASIC MODE: Provide simple explanations.",
          "if": "setting.mode == \"basic\""
        },
        {
          "content": "\nADVANCED MODE: Include technical details.",
          "if": "setting.mode == \"advanced\""
        },
        {
          "content": "\nEXPERT MODE: Assume deep technical knowledge.",
          "if": "setting.mode == \"expert\""
        }
      ]
    }
  ]
}
```

### Negation

```json
{
  "content": "\nWeb search disabled. Use knowledge base only.",
  "if": "setting.enableWebSearch == false"
}
```

## Real-World Examples

### E-Commerce Agent Settings

```json
{
  "settings": [
    {
      "name": "budgetMode",
      "title": "Budget-Conscious Shopping",
      "type": "bool",
      "description": "Prioritize value and cost-effective options",
      "defaultValue": false
    },
    {
      "name": "priceRange",
      "title": "Price Range",
      "type": "string",
      "description": "Preferred price range (budget: <$25, moderate: $25-100, premium: $100-500, luxury: $500+)",
      "defaultValue": "moderate"
    },
    {
      "name": "maxResults",
      "title": "Maximum Results",
      "type": "number",
      "description": "Maximum number of products to show per search",
      "defaultValue": 5
    },
    {
      "name": "includeOutOfStock",
      "title": "Include Out of Stock Items",
      "type": "bool",
      "description": "Show out-of-stock items in results",
      "defaultValue": false
    }
  ]
}
```

**Usage in instructions:**

```json
{
  "instructions": [
    {
      "content": "Search for products and provide recommendations..."
    },
    {
      "content": "\nBUDGET MODE:\n- Lead with most affordable options\n- Calculate per-unit pricing\n- Highlight sales and discounts\n- Suggest bulk purchase options",
      "if": "setting.budgetMode == true"
    },
    {
      "content": "\nPRICE FILTER: Budget\nFocus on products under $25. Emphasize value.",
      "if": "setting.priceRange == \"budget\""
    },
    {
      "content": "\nPRICE FILTER: Luxury\nFocus on premium brands. Price is secondary to quality.",
      "if": "setting.priceRange == \"luxury\""
    },
    {
      "content": "\nINCLUDE OUT OF STOCK:\nShow all matching products. Mark availability clearly.",
      "if": "setting.includeOutOfStock == true"
    }
  ]
}
```

### Content Creation Settings

```json
{
  "settings": [
    {
      "name": "contentLength",
      "title": "Target Content Length",
      "type": "string",
      "description": "Preferred content length (short: 300-500, medium: 750-1000, long: 1500-2000 words)",
      "defaultValue": "medium"
    },
    {
      "name": "tone",
      "title": "Writing Tone",
      "type": "string",
      "description": "Content tone (casual, professional, technical, friendly, formal)",
      "defaultValue": "professional"
    },
    {
      "name": "includeCitations",
      "title": "Include Citations",
      "type": "bool",
      "description": "Add source citations and references",
      "defaultValue": true
    },
    {
      "name": "seoOptimized",
      "title": "SEO Optimization",
      "type": "bool",
      "description": "Optimize content for search engines",
      "defaultValue": false
    }
  ]
}
```

### Customer Support Settings

```json
{
  "settings": [
    {
      "name": "autoEscalate",
      "title": "Auto-Escalate Urgent Issues",
      "type": "bool",
      "description": "Automatically flag tickets with urgent keywords",
      "defaultValue": true
    },
    {
      "name": "responseSLA",
      "title": "Response Time SLA (hours)",
      "type": "number",
      "description": "Target response time in hours",
      "defaultValue": 24
    },
    {
      "name": "supportLevel",
      "title": "Support Level",
      "type": "string",
      "description": "Support tier (basic, premium, enterprise)",
      "defaultValue": "basic"
    }
  ]
}
```

## Validation

### Type-Value Consistency

**Correct:**
```json
{"type": "bool", "defaultValue": true}
{"type": "string", "defaultValue": "moderate"}
{"type": "number", "defaultValue": 10}
```

**Incorrect:**
```json
{"type": "bool", "defaultValue": "true"}    // String, not boolean
{"type": "number", "defaultValue": "10"}    // String, not number
```

### Unique Names

Setting names must be unique within an agent:

```json
❌ {
  "settings": [
    {"name": "mode", "type": "string", ...},
    {"name": "mode", "type": "bool", ...}  // Duplicate!
  ]
}
```

### Required Fields

All five fields are required:

```json
❌ {
  "name": "enabled",
  "type": "bool",
  "defaultValue": true
  // Missing title and description
}
```

## Best Practices

### 1. Design for Users

**Good settings:**
- Clear purpose and impact
- Useful for multiple users
- Easy to understand
- Reasonable defaults

**Avoid:**
- Technical implementation details
- Settings rarely changed
- Overly granular options
- Settings with no impact

### 2. Limit Setting Count

**Optimal:** 3-7 settings per agent

**Too many settings:**
- Confuses users
- Increases complexity
- Harder to test
- Suggests refactoring needed

### 3. Group Related Settings

Organize settings logically:

```json
{
  "settings": [
    // Search behavior
    {"name": "enableWebSearch", ...},
    {"name": "maxResults", ...},
    
    // User preferences
    {"name": "saveHistory", ...},
    {"name": "personalizedResults", ...},
    
    // Output format
    {"name": "responseFormat", ...},
    {"name": "includeImages", ...}
  ]
}
```

### 4. Document Valid Values

For string settings, document options:

```json
{
  "description": "Output format. Valid values: markdown (formatted text), plain (no formatting), html (HTML markup), json (structured data)"
}
```

### 5. Use Sensible Defaults

```json
{
  "name": "notifications",
  "defaultValue": true    // ✓ Most users want notifications
}
```

```json
{
  "name": "experimentalFeatures",
  "defaultValue": false   // ✓ Experimental should be opt-in
}
```

## Testing Settings

### Test All Combinations

For an agent with 3 boolean settings, test all 8 combinations (2³):

```
enableWebSearch: true,  budgetMode: true,  saveHistory: true
enableWebSearch: true,  budgetMode: true,  saveHistory: false
enableWebSearch: true,  budgetMode: false, saveHistory: true
...
```

### Test Edge Cases

- Maximum/minimum numeric values
- Empty strings (if allowed)
- All string options
- Settings disabled/enabled together

### Test Defaults

Verify agent works correctly with all default values without modification.

## Troubleshooting

### Setting Not Taking Effect

**Check:**
1. Conditional expression syntax correct?
2. Setting name spelled correctly (case-sensitive)?
3. String values use escaped quotes in conditions?
4. Agent reloaded after changes?

### Type Mismatch Error

```
Error: settings[0]: type is "bool" but defaultValue is string
```

**Fix:** Ensure `defaultValue` matches `type`

### Duplicate Setting Name

```
Error: Agent.settings contains duplicate setting names: mode
```

**Fix:** Make all setting names unique within agent

## Next Steps

- **[Conditional Instructions](conditional-instructions.md)** - Master conditional logic
- **[Configuration Reference](../agents/configuration.md)** - Complete schema
- **[Examples](../agents/examples.md)** - See settings in action

---

**Related:**  
[Conditional Instructions](conditional-instructions.md) | [Agent Configuration](../agents/configuration.md) | [Creating Agents](../agents/creating-agents.md)

