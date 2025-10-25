# Creating Custom Agents

This guide walks you through creating production-ready AI agents step by step.

## Before You Begin

### What You'll Need

- Text editor for JSON files
- Access to `apps/api/src/agents/` directory
- Understanding of your agent's purpose and workflow
- List of required tools (optional)

### Planning Your Agent

Before writing code, answer these questions:

1. **What problem does this agent solve?**
2. **Is it a single task or complex workflow?**
3. **Does it need to maintain conversation context?**
4. **What external tools or data does it need?**
5. **Are there different modes or configurations?**

## Step-by-Step Guide

### Step 1: Define Agent Identity

Start with the basic structure:

```json
{
  "id": "your-agent-id",
  "name": "Your Agent Name",
  "description": "Clear description of what your agent does",
  "interactive": false,
  "settings": [],
  "agents": []
}
```

**Key decisions:**

**Agent ID** (`id`)
- Use lowercase with hyphens
- Make it descriptive: `grocery-assistant`, `report-generator`
- Must be unique across all agents

**Agent Name** (`name`)
- User-facing display name
- Can include spaces and capitals
- Should clearly indicate purpose

**Description** (`description`)
- Single sentence summary
- Used by switchboard for routing
- Focus on capabilities and use cases

**Interactive Mode** (`interactive`)
- `true`: Agent stays active across messages (conversations, iterative work)
- `false` or omitted: Returns to switchboard after each task (quick queries)

### Step 2: Design Your Personas

Decide if you need one or multiple personas:

#### Single Persona (Simple Agents)

For straightforward tasks with no workflow complexity:

```json
{
  "id": "currency-converter",
  "name": "Currency Converter",
  "description": "Converts between currencies using current exchange rates",
  "agents": [
    {
      "name": "converter",
      "description": "Performs currency conversions",
      "instructions": [
        {
          "content": "You convert currency amounts using current exchange rates. Format:\n- Input: Amount and currencies (e.g., '100 USD to EUR')\n- Output: Converted amount with rate and date\n- Always show calculation: '100 USD × 0.92 = 92 EUR (rate as of DATE)'\n\nBe precise with numbers and always specify the rate source."
        }
      ],
      "toolsets": ["duckduckgo"]
    }
  ]
}
```

#### Multiple Personas (Complex Workflows)

For tasks requiring specialization or sequential steps:

```json
{
  "id": "content-team",
  "name": "Content Creation Team",
  "description": "Creates professional content through coordinated research, writing, and editing",
  "interactive": true,
  "agents": [
    {
      "name": "project_manager",
      "description": "Coordinates the content creation workflow",
      "instructions": [
        {
          "content": "You manage a content creation team. For each request:\n1. Delegate to 'researcher' to gather information\n2. Delegate to 'writer' to create content\n3. Delegate to 'editor' for final review\n\nUse transfer_task to delegate. Present final result to user."
        }
      ],
      "subAgents": ["researcher", "writer", "editor"]
    },
    {
      "name": "researcher",
      "description": "Gathers and organizes information",
      "instructions": [
        {
          "content": "Research the requested topic thoroughly. Search for current, accurate information from reliable sources. Organize findings clearly with source citations. Focus on facts, statistics, and expert opinions."
        }
      ],
      "toolsets": ["duckduckgo"]
    },
    {
      "name": "writer",
      "description": "Creates polished content",
      "instructions": [
        {
          "content": "Write engaging content based on research provided. Structure:\n- Compelling introduction\n- Clear body sections with headers\n- Strong conclusion\n\nUse professional but accessible tone. Include concrete examples."
        }
      ]
    },
    {
      "name": "editor",
      "description": "Reviews and improves content",
      "instructions": [
        {
          "content": "Review content for:\n- Grammar and spelling\n- Clarity and flow\n- Accuracy and completeness\n- Engagement and readability\n\nMake improvements and note key changes made."
        }
      ]
    }
  ]
}
```

### Step 3: Write Effective Instructions

Instructions are the most critical part of your agent. They define behavior, constraints, and output format.

#### Instruction Structure

```json
{
  "content": "Your detailed instructions here..."
}
```

Or with conditions:

```json
{
  "content": "Instructions that apply when a setting is enabled...",
  "if": "setting.settingName == true"
}
```

#### Instruction Best Practices

**1. Define Role and Scope**

```json
{
  "content": "You are a technical support specialist for a SaaS platform. Your responsibilities:\n- Diagnose technical issues\n- Provide step-by-step troubleshooting\n- Link to relevant documentation\n- Escalate complex issues to engineering\n\nYou do NOT handle billing or account management questions."
}
```

**2. Specify Business Rules**

```json
{
  "content": "Business rules:\n- Never recommend out-of-stock items without mentioning alternatives\n- Always provide 3-5 options at different price points\n- Flag products with ratings below 4.0 stars\n- Include estimated delivery times\n- Highlight current promotions or sales"
}
```

**3. Define Output Format**

```json
{
  "content": "Format recommendations as:\n\n**Product Name** by Brand\n- Price: $XX.XX (Unit cost: $X.XX/oz)\n- Rating: ⭐ X.X (XXX reviews)\n- Key features: [bullet points]\n- Availability: In stock / Ships in X days\n\nLimit to 5 recommendations unless requested otherwise."
}
```

**4. Handle Edge Cases**

```json
{
  "content": "Error handling:\n- If product unavailable: Suggest 2-3 alternatives\n- If price unavailable: Note 'Price varies' and explain\n- If unclear request: Ask clarifying questions\n- If outside expertise: Politely redirect to appropriate resource"
}
```

### Step 4: Add Configuration Settings (Optional)

Settings make your agent flexible and reusable.

#### When to Add Settings

Add settings when you need to:
- Toggle features on/off
- Support different user segments
- Customize output formats
- Adjust behavior without code changes

#### Setting Types

**Boolean Settings** - For toggles

```json
{
  "name": "includeImages",
  "title": "Include Images",
  "type": "bool",
  "description": "Include product images in recommendations",
  "defaultValue": true
}
```

**String Settings** - For choices

```json
{
  "name": "sortOrder",
  "title": "Sort Order",
  "type": "string",
  "description": "Default sort order (relevance, price-low, price-high, rating)",
  "defaultValue": "relevance"
}
```

**Number Settings** - For thresholds

```json
{
  "name": "maxResults",
  "title": "Maximum Results",
  "type": "number",
  "description": "Maximum number of items to return",
  "defaultValue": 10
}
```

#### Using Settings in Instructions

Reference settings in conditional instructions:

```json
{
  "instructions": [
    {
      "content": "Base instructions that always apply..."
    },
    {
      "content": "\nBUDGET MODE: Prioritize value. Lead with store brands and economy options. Calculate per-unit pricing. Highlight sales and promotions.",
      "if": "setting.budgetMode == true"
    },
    {
      "content": "\nPREMIUM MODE: Focus on quality. Prioritize premium brands. Emphasize features and benefits over price.",
      "if": "setting.budgetMode == false"
    }
  ]
}
```

### Step 5: Integrate Tools

Tools extend your agent's capabilities by connecting to external systems.

#### Available Tools

- **`duckduckgo`** - Web search
- **`memory-server`** - Persistent data storage
- **`notification-server`** - Push notifications
- **`weather-tool`** - Weather information
- **`google-calendar`** - Calendar integration

See [Available Tools](../tools/available-tools.md) for complete list.

#### Adding Tools to Personas

```json
{
  "name": "product_finder",
  "description": "Searches for products",
  "instructions": [...],
  "toolsets": ["duckduckgo", "memory-server"]
}
```

#### Tool Selection Guidelines

- Add tools only to personas that need them
- Coordinator personas typically don't need tools (they delegate)
- Specialist personas use tools to complete their specific tasks
- Consider tool costs and rate limits

### Step 6: Validate and Test

#### Validation Checklist

Before deploying, verify:

- [ ] Valid JSON syntax (use a JSON validator)
- [ ] All required fields present (`id`, `name`, `description`, `agents`)
- [ ] Agent `id` is unique and follows naming convention
- [ ] Each persona has `name`, `description`, and `instructions`
- [ ] Boolean values are unquoted (`true`/`false`)
- [ ] Setting types match `defaultValue` types
- [ ] Setting names are unique within agent
- [ ] Conditional expressions use correct syntax
- [ ] Referenced toolsets exist in system
- [ ] SubAgent references point to valid personas

#### Testing Workflow

1. **Save** agent file to `apps/api/src/agents/your-agent.json`
2. **Restart** API service (if required for agent reload)
3. **Check logs** for any loading errors
4. **Test basic functionality** without settings
5. **Test each setting** configuration if applicable
6. **Test persona delegation** if multi-persona
7. **Test with various inputs** including edge cases

## Complete Example: E-Commerce Product Assistant

Here's a complete, production-ready agent:

```json
{
  "id": "product-assistant",
  "name": "Product Shopping Assistant",
  "description": "Helps customers find and compare products based on their needs and preferences",
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
      "description": "Preferred price range (budget, moderate, premium, luxury)",
      "defaultValue": "moderate"
    },
    {
      "name": "maxResults",
      "title": "Maximum Results",
      "type": "number",
      "description": "Maximum number of products to show",
      "defaultValue": 5
    }
  ],
  "agents": [
    {
      "name": "shopping_coordinator",
      "description": "Manages the product search and recommendation process",
      "model": "anthropic",
      "instructions": [
        {
          "content": "You coordinate product shopping assistance. When customers request product recommendations:\n\n1. Understand their needs, preferences, and constraints\n2. Delegate to 'product_searcher' to find options\n3. Present recommendations with clear rationale\n4. Answer follow-up questions\n5. Help with comparisons and decisions\n\nMaintain friendly, helpful tone throughout the conversation."
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
          "content": "You search for and evaluate products. For each request:\n\n**Search Process:**\n- Use web search to find current products\n- Focus on reputable retailers and brands\n- Check availability and pricing\n- Review customer ratings and reviews\n\n**Evaluation Criteria:**\n- Match to customer requirements\n- Value for money\n- Quality indicators (ratings, reviews, brand reputation)\n- Availability and shipping\n\n**Output Format:**\n\n**[Product Name]** by [Brand]\n- **Price:** $XX.XX ([per-unit if applicable])\n- **Rating:** ⭐ X.X ([review count] reviews)\n- **Key Features:**\n  • Feature 1\n  • Feature 2\n  • Feature 3\n- **Availability:** [In stock / Ships in X days]\n- **Why recommended:** [Brief rationale]\n\n**Rules:**\n- Provide exactly the configured maximum number of results\n- Always include pricing and availability\n- Flag products with <4.0 ratings\n- Note any current sales or promotions"
        },
        {
          "content": "\n**BUDGET MODE ACTIVE:**\n- Lead with most affordable options\n- Prioritize store brands and budget-friendly choices\n- Calculate and show per-unit pricing\n- Highlight sales, discounts, and bulk options\n- Note value proposition in rationale",
          "if": "setting.budgetMode == true"
        },
        {
          "content": "\n**PRICE RANGE FILTER: Budget**\n- Focus on entry-level and economy products\n- Typical range: Under $25 for small items, under $100 for larger items\n- Emphasize best value for money",
          "if": "setting.priceRange == \"budget\""
        },
        {
          "content": "\n**PRICE RANGE FILTER: Premium**\n- Focus on high-quality, established brands\n- Typical range: $100-500 for most items\n- Emphasize quality, features, and reliability",
          "if": "setting.priceRange == \"premium\""
        },
        {
          "content": "\n**PRICE RANGE FILTER: Luxury**\n- Focus on top-tier, luxury brands\n- Price is secondary to quality and prestige\n- Emphasize craftsmanship, exclusivity, and brand heritage",
          "if": "setting.priceRange == \"luxury\""
        }
      ],
      "toolsets": ["duckduckgo"]
    }
  ]
}
```

## Common Patterns and Templates

### Pattern 1: Single-Purpose Utility

```json
{
  "id": "utility-agent",
  "name": "Utility Agent",
  "description": "Performs specific utility function",
  "agents": [
    {
      "name": "processor",
      "description": "Processes requests",
      "instructions": [{"content": "Instructions..."}],
      "toolsets": ["relevant-tool"]
    }
  ]
}
```

### Pattern 2: Interactive Assistant

```json
{
  "id": "assistant-agent",
  "name": "Assistant Agent",
  "description": "Provides interactive assistance with context retention",
  "interactive": true,
  "agents": [
    {
      "name": "helper",
      "description": "Assists users",
      "instructions": [{"content": "Instructions..."}],
      "toolsets": ["tools-as-needed"]
    }
  ]
}
```

### Pattern 3: Workflow Orchestrator

```json
{
  "id": "workflow-agent",
  "name": "Workflow Agent",
  "description": "Coordinates multi-step workflow",
  "interactive": true,
  "agents": [
    {
      "name": "coordinator",
      "description": "Orchestrates workflow",
      "instructions": [{"content": "Coordination logic..."}],
      "subAgents": ["step1", "step2", "step3"]
    },
    {
      "name": "step1",
      "description": "First step",
      "instructions": [{"content": "Step 1 instructions..."}]
    },
    {
      "name": "step2",
      "description": "Second step",
      "instructions": [{"content": "Step 2 instructions..."}]
    },
    {
      "name": "step3",
      "description": "Third step",
      "instructions": [{"content": "Step 3 instructions..."}]
    }
  ]
}
```

## Troubleshooting

### Common Errors

**"Missing required field: id"**
- Ensure all required properties are present
- Check for typos in property names

**"interactive must be boolean"**
- Use `true` or `false`, not `"true"` or `"false"`
- Or omit the property entirely (defaults to false)

**"Setting type mismatch"**
- Ensure `defaultValue` matches the `type`
- Boolean setting needs boolean default
- String setting needs string default

**"Invalid conditional expression"**
- Use format: `setting.<name> == <value>`
- String values need escaped quotes: `"setting.mode == \"value\""`
- Boolean values don't need quotes: `setting.enabled == true`

**"Unknown toolset"**
- Verify toolset name spelling (case-sensitive)
- Check toolset is registered in system
- See [Available Tools](../tools/available-tools.md)

### Getting Help

1. **Validate JSON** - Use online JSON validator
2. **Check examples** - Compare to working agent examples
3. **Review logs** - Server logs show specific validation errors
4. **Check documentation** - Review relevant sections
5. **Contact support** - Reach out to your administrator

## Next Steps

- **[Configuration Reference](configuration.md)** - Complete schema documentation
- **[Examples](examples.md)** - More real-world examples
- **[Settings Guide](../advanced/settings.md)** - Master settings and conditionals
- **[Tools](../tools/overview.md)** - Learn about available tools

---

**Related:**  
[Agent Overview](overview.md) | [Examples](examples.md) | [Multi-Persona Workflows](../advanced/multi-persona.md)

