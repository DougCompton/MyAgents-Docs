# Conditional Instructions

Dynamic agent behavior based on runtime configuration.

## Overview

**Conditional instructions** allow you to modify agent behavior based on settings without changing code. Instructions with `if` clauses are evaluated at runtime and included only when conditions are met.

## Basic Syntax

### Structure

```json
{
  "content": "Instructions to apply when condition is true",
  "if": "setting.<name> <operator> <value>"
}
```

### Components

1. **content**: The instructions to apply
2. **if**: Boolean expression (condition)
3. **setting.**: Required prefix for setting references
4. **operator**: `==` (equals) or `!=` (not equals)
5. **value**: The value to compare against

## Operators

### Equality (`==`)

Check if setting equals a value:

```json
{
  "content": "\nFeature enabled behavior...",
  "if": "setting.enableFeature == true"
}
```

### Inequality (`!=`)

Check if setting does NOT equal a value:

```json
{
  "content": "\nFallback behavior when feature is disabled...",
  "if": "setting.enableFeature != true"
}
```

## Type-Specific Examples

### Boolean Conditions

```json
{
  "settings": [
    {
      "name": "verboseMode",
      "type": "bool",
      "title": "Verbose Mode",
      "description": "Provide detailed explanations",
      "defaultValue": false
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
          "content": "\nVERBOSE MODE: Provide detailed explanations with examples and step-by-step reasoning.",
          "if": "setting.verboseMode == true"
        },
        {
          "content": "\nCONCISE MODE: Keep responses brief and to the point.",
          "if": "setting.verboseMode == false"
        }
      ]
    }
  ]
}
```

### String Conditions

**Important:** String values require escaped quotes!

```json
{
  "settings": [
    {
      "name": "outputFormat",
      "type": "string",
      "title": "Output Format",
      "description": "Format for responses (markdown, plain, json)",
      "defaultValue": "markdown"
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
          "content": "\nOUTPUT: Use markdown formatting with headers, lists, and code blocks.",
          "if": "setting.outputFormat == \"markdown\""
        },
        {
          "content": "\nOUTPUT: Use plain text only. No formatting characters.",
          "if": "setting.outputFormat == \"plain\""
        },
        {
          "content": "\nOUTPUT: Structure responses as valid JSON objects.",
          "if": "setting.outputFormat == \"json\""
        }
      ]
    }
  ]
}
```

### Number Conditions

```json
{
  "settings": [
    {
      "name": "maxResults",
      "type": "number",
      "title": "Maximum Results",
      "description": "Maximum items to return",
      "defaultValue": 10
    }
  ],
  "agents": [
    {
      "name": "searcher",
      "instructions": [
        {
          "content": "Search and provide results..."
        },
        {
          "content": "\nBrief mode: Return top 3 most relevant results only.",
          "if": "setting.maxResults == 3"
        },
        {
          "content": "\nComprehensive mode: Provide extensive results with detailed analysis.",
          "if": "setting.maxResults == 20"
        }
      ]
    }
  ]
}
```

## Common Patterns

### Feature Flags

Enable/disable entire features:

```json
{
  "settings": [
    {
      "name": "enableWebSearch",
      "type": "bool",
      "title": "Enable Web Search",
      "description": "Allow searching the web for information",
      "defaultValue": true
    }
  ],
  "agents": [
    {
      "name": "assistant",
      "instructions": [
        {
          "content": "You are a helpful assistant..."
        },
        {
          "content": "\nWEB SEARCH ENABLED:\n- For current events, use web search\n- Always cite sources with URLs\n- Verify information across multiple sources\n- Note publication dates",
          "if": "setting.enableWebSearch == true"
        },
        {
          "content": "\nWEB SEARCH DISABLED:\n- Use only your knowledge base\n- Note that information may not be current\n- Suggest enabling web search for latest info",
          "if": "setting.enableWebSearch == false"
        }
      ],
      "toolsets": ["duckduckgo"]
    }
  ]
}
```

### Mode Switching

Different behavior modes:

```json
{
  "settings": [
    {
      "name": "userLevel",
      "type": "string",
      "title": "User Experience Level",
      "description": "User expertise (beginner, intermediate, expert)",
      "defaultValue": "intermediate"
    }
  ],
  "agents": [
    {
      "name": "tutor",
      "instructions": [
        {
          "content": "You are a programming tutor..."
        },
        {
          "content": "\nBEGINNER MODE:\n- Explain concepts from basics\n- Avoid jargon\n- Provide lots of examples\n- Check understanding frequently",
          "if": "setting.userLevel == \"beginner\""
        },
        {
          "content": "\nINTERMEDIATE MODE:\n- Assume basic knowledge\n- Introduce some technical terms\n- Balance theory with practice\n- Provide relevant examples",
          "if": "setting.userLevel == \"intermediate\""
        },
        {
          "content": "\nEXPERT MODE:\n- Assume deep technical knowledge\n- Use industry terminology\n- Focus on edge cases and optimization\n- Discuss tradeoffs and alternatives",
          "if": "setting.userLevel == \"expert\""
        }
      ]
    }
  ]
}
```

### Cascading Conditions

Layer multiple conditional instructions:

```json
{
  "settings": [
    {
      "name": "dietaryMode",
      "type": "string",
      "title": "Dietary Restrictions",
      "description": "Filter by diet (none, vegetarian, vegan, keto)",
      "defaultValue": "none"
    },
    {
      "name": "budgetMode",
      "type": "bool",
      "title": "Budget Mode",
      "description": "Prioritize affordable options",
      "defaultValue": false
    }
  ],
  "agents": [
    {
      "name": "meal_planner",
      "instructions": [
        {
          "content": "Suggest meals and recipes..."
        },
        {
          "content": "\nDIETARY: Vegetarian filter active. Exclude all meat, poultry, fish.",
          "if": "setting.dietaryMode == \"vegetarian\""
        },
        {
          "content": "\nDIETARY: Vegan filter active. Exclude all animal products.",
          "if": "setting.dietaryMode == \"vegan\""
        },
        {
          "content": "\nDIETARY: Keto filter active. Focus on low-carb, high-fat options.",
          "if": "setting.dietaryMode == \"keto\""
        },
        {
          "content": "\nBUDGET MODE: Prioritize affordable ingredients and recipes under $10/serving.",
          "if": "setting.budgetMode == true"
        }
      ]
    }
  ]
}
```

**Result when both active:**
```
Base instructions
+ Vegetarian filter
+ Budget mode
= Affordable vegetarian meals
```

## Best Practices

### 1. Always Include Base Instructions

Start with unconditional instructions:

```json
{
  "instructions": [
    {
      "content": "You are a helpful assistant. [Core behavior that always applies]"
    },
    {
      "content": "\n[Conditional behavior]",
      "if": "setting.mode == \"advanced\""
    }
  ]
}
```

### 2. Use Clear Markers

Make conditional sections obvious:

```json
{
  "content": "\n**BUDGET MODE ACTIVE:**\n- Prioritize value\n- Show per-unit pricing\n- Highlight sales",
  "if": "setting.budgetMode == true"
}
```

### 3. Handle All Cases

For mutually exclusive modes, cover all options:

```json
{
  "instructions": [
    {"content": "Base..."},
    {"content": "\nMode A behavior", "if": "setting.mode == \"a\""},
    {"content": "\nMode B behavior", "if": "setting.mode == \"b\""},
    {"content": "\nMode C behavior", "if": "setting.mode == \"c\""}
  ]
}
```

### 4. Keep Conditions Simple

**Good:**
```json
{"if": "setting.enabled == true"}
{"if": "setting.mode == \"advanced\""}
```

**Avoid complex logic:**
```json
// NOT SUPPORTED
{"if": "setting.a == true AND setting.b == false"}
{"if": "setting.value > 10"}
```

Use separate conditionals instead:

```json
{
  "content": "\nBoth A and B active",
  "if": "setting.enableA == true"
},
{
  "content": "\nAdditional B behavior",
  "if": "setting.enableB == true"
}
```

### 5. Test All Paths

Test agent with each condition:
- All settings at defaults
- Each setting toggled individually
- Common setting combinations
- Edge cases

## Common Errors

### Missing Quotes for Strings

```json
❌ "if": "setting.mode == vegan"        // Missing quotes
✅ "if": "setting.mode == \"vegan\""    // Correct
```

### Wrong Prefix

```json
❌ "if": "mode == true"                 // Missing "setting."
✅ "if": "setting.mode == true"         // Correct
```

### Wrong Operator

```json
❌ "if": "setting.value = 10"           // Single = (assignment)
✅ "if": "setting.value == 10"          // Double == (comparison)
```

### Invalid Boolean String

```json
❌ "if": "setting.enabled == \"true\""  // String "true"
✅ "if": "setting.enabled == true"      // Boolean true
```

### Using "if": "true"

```json
❌ {
  "content": "Always applies",
  "if": "true"    // Invalid syntax
}

✅ {
  "content": "Always applies"
  // Omit "if" entirely for unconditional instructions
}
```

## Advanced Techniques

### Instruction Composition

Build complex behavior from simple conditions:

```json
{
  "settings": [
    {"name": "includeImages", "type": "bool", ...},
    {"name": "includeVideos", "type": "bool", ...},
    {"name": "includeCharts", "type": "bool", ...}
  ],
  "agents": [
    {
      "name": "content_creator",
      "instructions": [
        {"content": "Create content..."},
        {
          "content": "\nINCLUDE IMAGES: Add relevant images with captions.",
          "if": "setting.includeImages == true"
        },
        {
          "content": "\nINCLUDE VIDEOS: Embed video content where appropriate.",
          "if": "setting.includeVideos == true"
        },
        {
          "content": "\nINCLUDE CHARTS: Create data visualizations for statistics.",
          "if": "setting.includeCharts == true"
        }
      ]
    }
  ]
}
```

### Tool Exclusions with Conditions

Conditionally disable tools:

```json
{
  "toolsets": [
    {
      "name": "memory-server",
      "exclude": [
        {"tool": "delete", "if": "setting.readOnly == true"},
        {"tool": "update", "if": "setting.readOnly == true"}
      ]
    }
  ]
}
```

## Debugging Conditionals

### Check Expression Syntax

1. Verify `setting.` prefix
2. Check operator (`==` or `!=`)
3. Verify value type matches setting type
4. For strings, check escaped quotes

### Test Individual Conditions

Test each conditional independently:

1. Set only one setting to non-default
2. Verify that conditional activates
3. Check behavior is correct
4. Repeat for each setting

### Review Logs

Server logs show which conditionals are evaluated and their results.

## Next Steps

- **[Settings Guide](settings.md)** - Master settings configuration
- **[Multi-Persona Workflows](multi-persona.md)** - Complex orchestration
- **[Examples](../agents/examples.md)** - See conditionals in action

---

**Related:**  
[Settings](settings.md) | [Agent Configuration](../agents/configuration.md) | [Creating Agents](../agents/creating-agents.md)

