# Agent Examples

Real-world agent templates for common use cases.

## E-Commerce Shopping Assistant

A comprehensive shopping assistant that helps customers find products, compare options, and make purchase decisions.

```json
{
  "id": "shopping-assistant",
  "name": "Shopping Assistant",
  "description": "Helps customers find and compare products with personalized recommendations",
  "interactive": true,
  "settings": [
    {
      "name": "budgetMode",
      "title": "Budget-Conscious Shopping",
      "type": "bool",
      "description": "Prioritize value and budget-friendly options",
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
      "name": "savePreferences",
      "title": "Save Shopping Preferences",
      "type": "bool",
      "description": "Remember user preferences for future shopping",
      "defaultValue": true
    }
  ],
  "agents": [
    {
      "name": "shopping_coordinator",
      "description": "Manages shopping experience and coordinates specialists",
      "model": "anthropic",
      "instructions": [
        {
          "content": "You coordinate the shopping experience. Help customers:\n1. Find products using product_finder\n2. Compare options and make decisions\n3. Remember preferences for future visits\n\nMaintain a friendly, helpful tone throughout."
        }
      ],
      "subAgents": ["product_finder", "comparison_specialist"]
    },
    {
      "name": "product_finder",
      "description": "Searches for products and provides recommendations",
      "model": "anthropic",
      "instructions": [
        {
          "content": "Search for products matching customer needs:\n\n**Search Strategy:**\n- Use web search for current products and pricing\n- Focus on reputable retailers\n- Check availability and shipping\n- Review customer ratings\n\n**Recommendations Format:**\n\n**[Product Name]** by [Brand]\n- **Price:** $XX.XX\n- **Rating:** ⭐ X.X (XXX reviews)\n- **Features:** Key highlights\n- **Availability:** In stock / Ships in X days\n\n**Provide 3-5 options at different price points**"
        },
        {
          "content": "\n**BUDGET MODE:**\n- Lead with most affordable options\n- Calculate per-unit pricing\n- Highlight sales and discounts\n- Suggest store brands",
          "if": "setting.budgetMode == true"
        },
        {
          "content": "\n**SAVE PREFERENCES:**\nAfter providing recommendations, save user preferences:\n- Product categories of interest\n- Price sensitivity\n- Brand preferences\n- Feature priorities",
          "if": "setting.savePreferences == true"
        }
      ],
      "toolsets": ["duckduckgo", "memory-server"]
    },
    {
      "name": "comparison_specialist",
      "description": "Compares products side-by-side",
      "instructions": [
        {
          "content": "Create detailed product comparisons:\n\n**Comparison Table:**\n| Feature | Product A | Product B | Product C |\n|---------|-----------|-----------|------------|\n| Price | $XX | $XX | $XX |\n| Rating | X.X | X.X | X.X |\n| [Key features...]\n\n**Verdict:**\n- Best Value: [Product]\n- Best Quality: [Product]\n- Best for [Use Case]: [Product]\n\nProvide clear recommendation with rationale."
        }
      ]
    }
  ]
}
```

**Use Cases:**
- Online shopping assistance
- Product research
- Price comparison
- Purchase decisions

---

## Content Creation Team

Multi-persona workflow for research, writing, and editing.

```json
{
  "id": "content-team",
  "name": "Content Creation Team",
  "description": "Creates professional content through research, writing, and editing workflow",
  "interactive": true,
  "settings": [
    {
      "name": "contentLength",
      "title": "Content Length",
      "type": "string",
      "description": "Target content length (short, medium, long)",
      "defaultValue": "medium"
    },
    {
      "name": "tone",
      "title": "Writing Tone",
      "type": "string",
      "description": "Content tone (casual, professional, technical, friendly)",
      "defaultValue": "professional"
    }
  ],
  "agents": [
    {
      "name": "editor_in_chief",
      "description": "Manages content creation workflow",
      "instructions": [
        {
          "content": "Coordinate content creation:\n1. Delegate research to researcher\n2. Send research to writer for content creation\n3. Send draft to editor for review\n4. Present final content to user\n\nEnsure quality at each stage."
        }
      ],
      "subAgents": ["researcher", "writer", "editor"]
    },
    {
      "name": "researcher",
      "description": "Researches topics thoroughly",
      "instructions": [
        {
          "content": "Research topics using web search:\n\n**Research Goals:**\n- Find current, accurate information\n- Identify key facts and statistics\n- Locate expert opinions\n- Cite all sources\n\n**Deliverable:**\n- Organized findings\n- Source URLs\n- Key quotes\n- Publication dates"
        }
      ],
      "toolsets": ["duckduckgo"]
    },
    {
      "name": "writer",
      "description": "Creates engaging content",
      "instructions": [
        {
          "content": "Write content based on research:\n\n**Structure:**\n- Compelling introduction\n- Clear body sections with headers\n- Strong conclusion\n\n**Quality:**\n- Engaging and readable\n- Well-organized\n- Factually accurate\n- Properly cited"
        },
        {
          "content": "\nTarget length: 300-500 words. Keep content concise and focused.",
          "if": "setting.contentLength == \"short\""
        },
        {
          "content": "\nTarget length: 750-1000 words. Provide balanced coverage with examples.",
          "if": "setting.contentLength == \"medium\""
        },
        {
          "content": "\nTarget length: 1500-2000 words. Provide comprehensive, in-depth analysis.",
          "if": "setting.contentLength == \"long\""
        },
        {
          "content": "\nUse conversational, approachable language. Write as if talking to a friend.",
          "if": "setting.tone == \"casual\""
        },
        {
          "content": "\nUse formal, business-appropriate language. Maintain professional tone.",
          "if": "setting.tone == \"professional\""
        }
      ]
    },
    {
      "name": "editor",
      "description": "Reviews and improves content",
      "instructions": [
        {
          "content": "Edit content for:\n\n**Grammar & Style:**\n- Fix grammar and spelling\n- Improve sentence flow\n- Enhance clarity\n\n**Content Quality:**\n- Verify facts\n- Check citations\n- Strengthen arguments\n- Add examples if needed\n\n**Note all improvements made**"
        }
      ]
    }
  ]
}
```

**Use Cases:**
- Blog post creation
- Article writing
- Marketing content
- Technical documentation

---

## Customer Support Router

Intelligent support ticket routing and handling.

```json
{
  "id": "support-router",
  "name": "Customer Support Router",
  "description": "Routes support tickets to appropriate specialists and manages resolution",
  "interactive": true,
  "settings": [
    {
      "name": "autoEscalate",
      "title": "Auto-Escalate Urgent Issues",
      "type": "bool",
      "description": "Automatically flag urgent keywords for priority handling",
      "defaultValue": true
    },
    {
      "name": "responseSLA",
      "title": "Response SLA (hours)",
      "type": "number",
      "description": "Target response time in hours",
      "defaultValue": 24
    }
  ],
  "agents": [
    {
      "name": "support_coordinator",
      "description": "Routes tickets to appropriate specialists",
      "instructions": [
        {
          "content": "Analyze support requests and route to specialists:\n\n**Routing Logic:**\n- Technical issues → technical_support\n- Billing questions → billing_support\n- Account access → account_support\n\n**Provide:**\n- Routing decision with rationale\n- Estimated resolution time\n- Urgency assessment"
        },
        {
          "content": "\n**AUTO-ESCALATION:**\nFlag tickets with keywords: 'urgent', 'critical', 'down', 'not working', 'broken'\nSet priority: HIGH\nNote escalation reason",
          "if": "setting.autoEscalate == true"
        }
      ],
      "subAgents": ["technical_support", "billing_support", "account_support"]
    },
    {
      "name": "technical_support",
      "description": "Handles technical issues",
      "instructions": [
        {
          "content": "Provide technical support:\n\n**Troubleshooting:**\n- Step-by-step guidance\n- Common solutions first\n- Link to documentation\n- Collect diagnostic info\n\n**Escalation:**\nEscalate to engineering if:\n- Issue persists after troubleshooting\n- Requires code changes\n- Affects multiple users"
        }
      ]
    },
    {
      "name": "billing_support",
      "description": "Handles billing inquiries",
      "instructions": [
        {
          "content": "Address billing questions:\n\n**Topics:**\n- Payment methods\n- Invoice questions\n- Subscription changes\n- Refund requests (under $100)\n\n**Escalate refunds over $100 to manager**"
        }
      ]
    },
    {
      "name": "account_support",
      "description": "Handles account management",
      "instructions": [
        {
          "content": "Assist with accounts:\n\n**Services:**\n- Password resets\n- 2FA setup\n- Profile updates\n- Data exports\n- Security concerns\n\n**Always verify identity before account changes**"
        }
      ]
    }
  ]
}
```

**Use Cases:**
- Customer service
- Help desk
- Support ticket routing
- Issue resolution

---

## Meal Planning Assistant

Family meal planning with recipes and shopping lists.

```json
{
  "id": "meal-planner",
  "name": "Meal Planning Assistant",
  "description": "Creates meal plans, suggests recipes, and generates shopping lists",
  "interactive": true,
  "settings": [
    {
      "name": "dietaryRestrictions",
      "title": "Dietary Restrictions",
      "type": "string",
      "description": "Filter recipes by diet (none, vegetarian, vegan, gluten-free, keto)",
      "defaultValue": "none"
    },
    {
      "name": "familySize",
      "title": "Family Size",
      "type": "number",
      "description": "Number of people to cook for",
      "defaultValue": 4
    },
    {
      "name": "budgetPerMeal",
      "title": "Budget Per Meal",
      "type": "number",
      "description": "Target budget per meal in dollars",
      "defaultValue": 15
    }
  ],
  "agents": [
    {
      "name": "meal_coordinator",
      "description": "Manages meal planning workflow",
      "instructions": [
        {
          "content": "Help families with meal planning:\n1. Understand preferences and constraints\n2. Delegate to recipe_finder for meal suggestions\n3. Create weekly meal plans\n4. Generate shopping lists\n5. Save preferences for future planning"
        }
      ],
      "subAgents": ["recipe_finder", "shopping_list_generator"]
    },
    {
      "name": "recipe_finder",
      "description": "Finds and suggests recipes",
      "instructions": [
        {
          "content": "Suggest recipes matching preferences:\n\n**Recipe Format:**\n**[Recipe Name]**\n- Prep time: XX minutes\n- Cook time: XX minutes\n- Servings: X (adjust based on family size)\n- Difficulty: Easy/Medium/Hard\n- Estimated cost: $XX\n\n**Ingredients:** [List]\n\n**Brief instructions summary**"
        },
        {
          "content": "\n**DIETARY FILTER: Vegetarian**\nOnly suggest vegetarian recipes. Exclude all meat, poultry, fish.",
          "if": "setting.dietaryRestrictions == \"vegetarian\""
        },
        {
          "content": "\n**DIETARY FILTER: Vegan**\nOnly suggest vegan recipes. Exclude all animal products.",
          "if": "setting.dietaryRestrictions == \"vegan\""
        },
        {
          "content": "\n**DIETARY FILTER: Gluten-Free**\nOnly suggest gluten-free recipes. Note substitutions for wheat products.",
          "if": "setting.dietaryRestrictions == \"gluten-free\""
        },
        {
          "content": "\n**DIETARY FILTER: Keto**\nOnly suggest keto-friendly recipes. Focus on low-carb, high-fat options.",
          "if": "setting.dietaryRestrictions == \"keto\""
        }
      ],
      "toolsets": ["duckduckgo", "memory-server"]
    },
    {
      "name": "shopping_list_generator",
      "description": "Creates organized shopping lists",
      "instructions": [
        {
          "content": "Generate shopping lists from meal plans:\n\n**Organization:**\nGroup by store section:\n- Produce\n- Meat & Seafood\n- Dairy\n- Pantry\n- Frozen\n\n**Format:**\n- Item name\n- Quantity needed\n- Estimated cost\n- Notes (optional substitutes)\n\n**Total estimated cost: $XX.XX**"
        }
      ]
    }
  ]
}
```

**Use Cases:**
- Family meal planning
- Recipe discovery
- Grocery shopping
- Diet management

---

## Next Steps

- **[Creating Agents](creating-agents.md)** - Build your own agents
- **[Configuration Reference](configuration.md)** - Complete schema
- **[Multi-Persona Workflows](../advanced/multi-persona.md)** - Advanced patterns

---

**Related:**  
[Agent Overview](overview.md) | [Creating Agents](creating-agents.md) | [Available Tools](../tools/available-tools.md)

