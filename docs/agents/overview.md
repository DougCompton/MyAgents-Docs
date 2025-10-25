# Understanding Agents

This guide explains the core concepts and architecture of AI agents in the MyAgents platform.

## What is an Agent?

An **agent** is a specialized AI assistant designed to handle specific business workflows or domains. Agents combine:

- **Domain Expertise** - Specialized knowledge and context for specific tasks
- **Personas** - Multiple specialized roles that work together
- **Tools** - Integration with external systems and services via MCP
- **Configuration** - Runtime behavior customization through settings

## Agent Architecture

### Core Components

Every agent consists of four main components:

```
┌─────────────────────────────────────┐
│           Agent                     │
├─────────────────────────────────────┤
│  Identity (id, name, description)   │
│  Behavior Mode (interactive)        │
│  Configuration (settings)           │
│  Personas (specialized roles)       │
└─────────────────────────────────────┘
```

#### 1. Identity
Defines who the agent is and what it does:
- **`id`**: Unique system identifier (e.g., `"grocery-assistant"`)
- **`name`**: User-facing display name (e.g., `"Grocery Shopping Assistant"`)
- **`description`**: Brief summary of capabilities for routing

#### 2. Behavior Mode
Controls conversation flow:
- **`interactive: true`**: Agent maintains context across messages
- **`interactive: false`** or omitted: Agent returns to switchboard after completing task

#### 3. Configuration
Runtime customization through settings:
- Boolean toggles for feature flags
- String options for modes and formats
- Number values for thresholds and limits

#### 4. Personas
Specialized roles that define agent capabilities:
- Each persona has specific instructions and tools
- Personas can delegate to sub-personas
- Enables complex workflow orchestration

## Agent Patterns

### Single-Persona Agent

The simplest pattern - one persona handles all interactions.

```json
{
  "id": "weather-agent",
  "name": "Weather Agent",
  "description": "Provides current weather information",
  "agents": [
    {
      "name": "weather_helper",
      "description": "Retrieves and presents weather data",
      "instructions": [
        {
          "content": "You provide weather information. Use the weather tool to get current conditions and forecasts. Present information clearly and concisely."
        }
      ],
      "toolsets": ["weather-tool"]
    }
  ]
}
```

**When to use:**
- Simple, focused tasks
- Single-step operations
- Specialized utilities

**Examples:**
- Weather lookups
- Currency conversion
- Simple calculations
- Quick information retrieval

### Multi-Persona Agent

Multiple specialized personas work together to handle complex workflows.

```json
{
  "id": "support-agent",
  "name": "Customer Support Agent",
  "description": "Routes and handles customer support inquiries",
  "interactive": true,
  "agents": [
    {
      "name": "coordinator",
      "description": "Routes requests to appropriate specialists",
      "instructions": [{
        "content": "Analyze customer requests and delegate to:\n- technical_support for bugs and technical issues\n- billing_support for payment questions\n- account_support for account access\n\nUse transfer_task to delegate work."
      }],
      "subAgents": ["technical_support", "billing_support", "account_support"]
    },
    {
      "name": "technical_support",
      "description": "Handles technical issues",
      "instructions": [{
        "content": "Provide technical troubleshooting. Offer step-by-step guidance and link to documentation."
      }]
    },
    {
      "name": "billing_support",
      "description": "Handles billing inquiries",
      "instructions": [{
        "content": "Address payment and subscription questions. Provide invoice details and handle refund requests."
      }]
    },
    {
      "name": "account_support",
      "description": "Handles account management",
      "instructions": [{
        "content": "Assist with account settings, password resets, and security concerns."
      }]
    }
  ]
}
```

**When to use:**
- Complex, multi-step workflows
- Task requiring specialization
- Decision trees with routing
- Sequential processing pipelines

**Examples:**
- Content creation teams (research → write → edit)
- Customer support routing
- Order processing (validate → process → fulfill)
- Report generation (collect → analyze → format)

## Persona Design

### Persona Anatomy

Each persona has distinct capabilities and responsibilities:

```json
{
  "name": "persona_identifier",
  "description": "Brief description of role",
  "model": "anthropic",
  "instructions": [
    {"content": "Detailed behavioral instructions"}
  ],
  "subAgents": ["other_persona"],
  "toolsets": ["tool1", "tool2"]
}
```

### Required Properties

#### `name` (string)
Unique identifier within the agent scope.

**Naming conventions:**
- Use lowercase with underscores: `product_specialist`
- Choose descriptive, role-based names
- Keep under 30 characters
- Must be unique within the agent

#### `description` (string)
Concise summary of persona responsibilities.

**Guidelines:**
- Single sentence, under 100 characters
- Focus on primary function
- Use active voice

#### `instructions` (array)
Defines persona behavior and operational guidelines.

**Best practices:**
- Start with role definition and scope
- Specify business rules and constraints
- Define output format expectations
- Use conditional instructions for configurability

### Optional Properties

#### `model` (string)
Specifies which LLM provider to use.

**Available models:**
- `"anthropic"` - Claude (best for analysis, writing, complex reasoning)
- `"gpt-4o"` - OpenAI GPT-4 (general purpose, versatile)

**When to specify:**
- Optimize performance for specific task types
- Manage costs (different models have different pricing)
- Quality requirements (some models excel at certain tasks)

#### `subAgents` (array of strings)
Lists personas this persona can delegate to.

**Use for:**
- Coordinator personas managing workflows
- Hierarchical task decomposition
- Specialized routing based on request type

#### `toolsets` (array)
Defines external tool access.

**Common toolsets:**
- `duckduckgo` - Web search
- `weather-tool` - Weather data
- `memory-server` - Persistent storage
- `notification-server` - Push notifications
- `google-calendar` - Calendar integration

## Interactive vs Non-Interactive Agents

The `interactive` property controls agent persistence across messages.

### Interactive Mode (`"interactive": true`)

Agent remains active after processing each message.

**Use cases:**
- Multi-turn conversations
- Iterative work (editing, refinement)
- Context-dependent tasks
- Shopping sessions

**User experience:**
```
User: "Create a blog post about AI"
Agent: [creates blog post]
(Agent remains active)
User: "Make it shorter"
Agent: [continues with same context]
```

**Examples:**
- E-commerce shopping assistants
- Content creation and editing
- Research and analysis
- Customer service conversations

### Non-Interactive Mode (default)

Agent completes task and returns to switchboard.

**Use cases:**
- Quick queries
- Single-transaction tasks
- Automated workflows
- Simple lookups

**User experience:**
```
User: "What's the weather?"
Agent: [provides weather]
(Returns to switchboard)
User: "Thanks"
Switchboard: [handles next request]
```

**Examples:**
- Weather lookups
- Currency conversion
- Simple calculations
- Quick fact retrieval

## Agent Lifecycle

### 1. Definition
Agent is defined in JSON configuration file in `apps/api/src/agents/`.

### 2. Loading
System loads and validates agent configuration at startup.

**Validation checks:**
- Required fields present
- JSON syntax correct
- Settings properly configured
- Toolsets available
- Conditional expressions valid

### 3. Registration
Agent registers with switchboard for routing.

**Switchboard uses:**
- Agent `description` for routing decisions
- Agent `id` for direct invocation
- Agent `name` for user display

### 4. Execution
Agent processes user requests through active persona.

**Execution flow:**
1. User sends message
2. Switchboard routes to appropriate agent (if needed)
3. Agent's active persona processes request
4. Persona executes instructions and uses tools
5. Persona may delegate to sub-personas
6. Response returned to user
7. Agent stays active (interactive) or returns to switchboard

### 5. Switching
Users can switch agents explicitly or via switchboard routing.

**Switching mechanisms:**
- **Automatic**: Switchboard routes based on request content
- **Explicit**: User requests specific agent
- **Delegation**: Persona uses `transfer_task` to delegate to sub-persona

## Switchboard Agent

The **switchboard** is a special system agent that coordinates all agent interactions.

### Responsibilities

1. **Route Requests** - Direct user messages to appropriate specialized agents
2. **Agent Discovery** - Analyze available agents and their capabilities
3. **Context Management** - Maintain conversation context during agent switches
4. **Fallback Handling** - Handle requests when no specialized agent is appropriate

### How Switchboard Routing Works

The switchboard uses agent descriptions to make routing decisions:

```json
{
  "id": "blog-team",
  "name": "Blog Writing Team",
  "description": "Coordinates research, writing, and editing for blog content creation"
}
```

When a user says: "I need help writing a blog post"

The switchboard:
1. Analyzes the user's request
2. Reviews available agent descriptions
3. Identifies best match based on capabilities
4. Routes request using `switch_agent` tool

### User Experience

From the user perspective:
```
User: "I want to write a blog post about AI"
→ Switchboard routes to Blog Writing Team

User: "What's the weather like?"
→ (If Blog Team is interactive and active)
→ Blog Team returns to switchboard
→ Switchboard routes to Weather Agent
```

## Agent Best Practices

### Naming Conventions

**Agent IDs:**
- ✅ `grocery-assistant`, `report-generator`, `customer-service`
- ❌ `ga`, `rep`, `Grocery Assistant` (too short, spaces, capitals)

**Agent Names:**
- ✅ `"Grocery Shopping Assistant"`, `"Business Report Generator"`
- ❌ `"GA"`, `"agent"` (not descriptive)

**Persona Names:**
- ✅ `product_specialist`, `data_analyst`, `content_writer`
- ❌ `ps`, `analyst1`, `writer-agent` (too short, inconsistent)

### Description Writing

Write clear, concise descriptions that help routing:

**Good descriptions:**
- ✅ "Assists customers with grocery shopping, product recommendations, and meal planning"
- ✅ "Routes customer support tickets to appropriate specialists based on issue type"
- ✅ "Generates business analytics reports with metrics and insights"

**Poor descriptions:**
- ❌ "Agent for grocery stuff"
- ❌ "Helps users"
- ❌ "" (empty)

### Instruction Design

Create specific, actionable instructions:

**Good instructions:**
```json
{
  "content": "You are a product specialist. Responsibilities:\n- Search catalog for products matching customer needs\n- Provide 3-5 recommendations with pricing and features\n- Include alternatives at different price points\n- Highlight products with 4+ star ratings\n\nFormat recommendations as numbered lists with clear rationale."
}
```

**Poor instructions:**
```json
{
  "content": "Help with products."
}
```

### Tool Selection

Choose appropriate tools for each persona:

- **Information gathering**: `duckduckgo` for web search
- **Data persistence**: `memory-server` for storing user data
- **Notifications**: `notification-server` for alerts
- **Calendar**: `google-calendar` for scheduling
- **Delegation**: `transfer_task` automatically available to coordinators

## Common Patterns

### Coordinator Pattern
One persona orchestrates workflow by delegating to specialists.

```
Coordinator
├── Specialist 1 (with tools)
├── Specialist 2 (with tools)
└── Specialist 3 (with tools)
```

### Sequential Processing
Tasks flow through defined sequence.

```
Request → Research → Write → Edit → Deliver
```

### Parallel Specialists
Multiple specialists handle different aspects simultaneously.

```
              ┌── Technical Support
Request ──────┼── Billing Support
              └── Account Support
```

### Hierarchical Delegation
Multi-level coordination for complex workflows.

```
Project Manager
├── Research Team
│   ├── Web Researcher
│   └── Data Analyst
└── Writing Team
    ├── Writer
    └── Editor
```

## Next Steps

Now that you understand agent architecture:

- **[Creating Agents](creating-agents.md)** - Step-by-step creation guide
- **[Configuration Reference](configuration.md)** - Complete configuration options
- **[Examples](examples.md)** - Real-world agent templates
- **[Tools Overview](../tools/overview.md)** - Learn about available tools

---

**Related:**  
[Getting Started](../getting-started.md) | [Configuration Reference](configuration.md) | [Multi-Persona Workflows](../advanced/multi-persona.md)

