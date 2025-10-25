# Team Configuration Schema

Complete technical schema reference for agent team YAML files.

## Quick Reference

```yaml
version: "1"                          # Required: Schema version
id: team-id                           # Required: Unique team identifier
name: Team Name                       # Required: Display name
description: Team description         # Required: Routing description
interactive: false                    # Optional: Persistence mode
default_agent: agent_name             # Required: Entry point agent

settings:                             # Optional: User settings
  - name: setting_name                # Required: Setting identifier
    type: boolean | string | number   # Required: Data type
    title: Display Label              # Required: UI label
    description: Setting description   # Required: Help text
    defaultValue: value               # Required: Default value

agents:                               # Required: Agent definitions
  agent_name:                         # Required: Agent identifier
    type: llm | parallel | loop | sequence  # Optional: Agent type (default: llm)
    name: Display Name                # Required: Display name (for agents map key)
    description: Agent description    # Required: Agent role
    model: anthropic | gpt-4o         # Optional: LLM provider
    max_iterations: N                 # Required for loop type: 1-10
    instructions:                     # Required: Behavior instructions
      - "condition": Instructions     # Optional: Conditional instructions
      - "": Default instructions      # Required: Default fallback
    sub_agents: [...]                 # Optional: Delegatable agents
    toolsets: [...]                   # Optional: Available tools (llm only)
```

## Team-Level Schema

### version

**Type:** `string`  
**Required:** Yes  
**Valid Values:** `"1"`

Schema version for forward compatibility.

```yaml
version: "1"
```

---

### id

**Type:** `string`  
**Required:** Yes  
**Pattern:** `^[a-z][a-z0-9-]{2,49}$`  
**Length:** 3-50 characters

Unique identifier for the team.

**Rules:**
- Must start with lowercase letter
- Can contain: lowercase letters, numbers, hyphens
- No uppercase, no spaces, no underscores
- Must be unique across all teams

**Examples:**
```yaml
# Valid
id: weather-team
id: customer-support
id: blog-writer-v2

# Invalid
id: Weather-Team    # No uppercase
id: wt              # Too short
id: weather_team    # No underscores
```

---

### name

**Type:** `string`  
**Required:** Yes  
**Length:** 5-100 characters

User-facing display name.

**Rules:**
- Can include spaces and special characters
- Used in UI and chat interface
- Should be descriptive and user-friendly

**Examples:**
```yaml
name: Weather Assistant
name: Customer Support Team
name: Blog Writing & Editing
```

---

### description

**Type:** `string`  
**Required:** Yes  
**Length:** 20-500 characters

Brief summary of team capabilities used by switchboard for routing.

**Rules:**
- Describe what problems the team solves
- Use action-oriented language
- Focus on user benefits
- Critical for routing decisions

**Examples:**
```yaml
description: Provides current weather conditions, forecasts, and severe weather alerts for any location worldwide

description: Routes customer support inquiries to specialized agents handling technical issues, billing questions, and account management
```

---

### default_agent

**Type:** `string`  
**Required:** Yes

Name of the entry-point agent. Must match an agent name in the `agents` section.

**Rules:**
- Must be valid agent key from `agents` map
- Case-sensitive
- This agent receives initial user requests

**Example:**
```yaml
default_agent: coordinator

agents:
  coordinator:    # Must match
    # Configuration...
```

---

### interactive

**Type:** `boolean`  
**Required:** No  
**Default:** `false`

Controls whether team persists after processing messages.

**Values:**
- `true`: Team remains active for multi-turn conversations
- `false`: Team returns to switchboard after each response

**Example:**
```yaml
# Persistent team for shopping experience
interactive: true

# One-shot team for quick lookup
interactive: false
```

---

### settings

**Type:** `array` of setting objects  
**Required:** No

Runtime configuration options accessible via conditional instructions.

**Setting Object Schema:**

```yaml
settings:
  - name: string                      # Required: Unique identifier
    type: boolean | string | number   # Required: Data type
    title: string                     # Required: UI display label
    description: string               # Required: Help text
    defaultValue: value               # Required: Must match type
```

**Setting Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Unique setting identifier (camelCase or snake_case) |
| `type` | enum | Yes | Data type: `"boolean"`, `"string"`, or `"number"` |
| `title` | string | Yes | User-facing display name |
| `description` | string | Yes | Explanation of what setting controls |
| `defaultValue` | varies | Yes | Default value matching specified type |

**Examples:**

**Boolean setting:**
```yaml
settings:
  - name: verbose_mode
    type: boolean
    title: Verbose Output
    description: Include detailed explanations in responses
    defaultValue: false
```

**String setting:**
```yaml
settings:
  - name: output_format
    type: string
    title: Output Format
    description: Format for generated content
    defaultValue: markdown
```

**Number setting:**
```yaml
settings:
  - name: max_results
    type: number
    title: Maximum Results
    description: Number of results to return
    defaultValue: 10
```

---

### agents

**Type:** `object` (map of agent name to agent configuration)  
**Required:** Yes

Defines all agents in the team.

**Rules:**
- Must contain at least one agent
- Agent keys must be unique
- `default_agent` must reference one of these agents

**Example:**
```yaml
agents:
  coordinator:
    # Agent configuration
  
  specialist_1:
    # Agent configuration
  
  specialist_2:
    # Agent configuration
```

---

## Agent-Level Schema

### type

**Type:** `string`  
**Required:** No  
**Default:** `"llm"`  
**Valid Values:** `"llm"`, `"parallel"`, `"loop"`, `"sequence"`

Specifies agent execution pattern.

**Type Descriptions:**

| Type | Description | Requirements |
|------|-------------|--------------|
| `llm` | Standard interactive agent | None |
| `parallel` | Concurrent execution of sub-agents | Requires 2+ sub_agents |
| `loop` | Iterative refinement | Requires exactly 1 sub_agent + max_iterations |
| `sequence` | Sequential pipeline | Requires 2+ sub_agents |

**Examples:**
```yaml
# LLM agent (default)
agent1:
  type: llm

# Parallel agent
agent2:
  type: parallel
  sub_agents: [a, b, c]

# Loop agent
agent3:
  type: loop
  max_iterations: 3
  sub_agents: [refiner]

# Sequence agent
agent4:
  type: sequence
  sub_agents: [stage1, stage2, stage3]
```

---

### name

**Type:** `string`  
**Required:** Yes  
**Pattern:** `^[a-z][a-z0-9_]{2,49}$`  
**Length:** 3-50 characters

Unique identifier for the agent within team scope.

**Rules:**
- Must start with lowercase letter
- Can contain: lowercase letters, numbers, underscores
- Use snake_case convention
- Must be unique within team

**Examples:**
```yaml
# Valid
product_specialist
technical_support
content_writer

# Invalid
ProductSpecialist    # No camelCase
ps                   # Too short
agent-1              # No hyphens
```

---

### description

**Type:** `string`  
**Required:** Yes  
**Length:** 10-200 characters

Brief summary of agent's role and responsibilities.

**Rules:**
- One sentence preferred
- Focus on primary responsibility
- Use active voice

**Examples:**
```yaml
description: Conducts web research and gathers authoritative sources

description: Creates engaging, SEO-optimized blog content

description: Routes support tickets to appropriate specialists
```

---

### model

**Type:** `string`  
**Required:** No  
**Valid Values:** `"anthropic"`, `"gpt-4o"`

Specifies which LLM provider to use for this agent.

**Models:**
- `anthropic`: Claude models (good for writing, analysis)
- `gpt-4o`: GPT-4 models (general purpose)

**Example:**
```yaml
analyst:
  model: anthropic

writer:
  model: gpt-4o
```

---

### max_iterations

**Type:** `number`  
**Required:** Yes (for `loop` agents only)  
**Valid Range:** 1-10

Number of iterations for loop-type agents.

**Rules:**
- Required if `type: loop`
- Must be between 1 and 10
- Each iteration is a full agent execution

**Example:**
```yaml
refiner:
  type: loop
  max_iterations: 3
  sub_agents: [content_improver]
```

---

### instructions

**Type:** `array` of instruction objects  
**Required:** Yes

Defines agent behavior through conditional and default instructions.

**Instruction Object Schema:**

```yaml
instructions:
  - "condition": |
      Instructions when condition evaluates to true
  - "": |
      Default instructions (condition is empty string)
```

**Rules:**
- Must have at least one instruction with empty string `""` condition (default)
- Instructions evaluated in order from top to bottom
- First matching condition wins
- Use `|` for multiline strings

**Condition Syntax:**

**Boolean checks:**
```yaml
- "settings.verbose": Instructions when true
- "!settings.verbose": Instructions when false
```

**Equality:**
```yaml
- "settings.format == 'json'": Instructions for JSON
- "settings.format != 'json'": Instructions for non-JSON
```

**Comparisons:**
```yaml
- "settings.max_results > 20": Instructions for large result sets
- "settings.price_limit <= 50": Instructions for budget searches
```

**Logical operators:**
```yaml
- "settings.expert && settings.verbose": AND condition
- "settings.format == 'markdown' || settings.format == 'html'": OR condition
- "(settings.a || settings.b) && !settings.c": Complex logic
```

**Examples:**

**Simple:**
```yaml
instructions:
  - |
      You are a helpful assistant. Provide clear, accurate responses.
```

**Conditional:**
```yaml
instructions:
  - "settings.expert_mode": |
      Expert mode: Provide technical details and advanced concepts.
  
  - |
      Standard mode: Use accessible language and explain clearly.
```

**Multiple conditions:**
```yaml
instructions:
  - "settings.mode == 'expert' && settings.verbose": |
      Expert + Verbose: Comprehensive technical detail
  
  - "settings.mode == 'expert'": |
      Expert: Technical focus, concise
  
  - "settings.verbose": |
      Verbose: Detailed explanations, standard level
  
  - |
      Standard: Balanced responses
```

---

### sub_agents

**Type:** `array` of strings  
**Required:** Depends on agent type

Lists agents this agent can delegate to or execute.

**Requirements by Type:**

| Agent Type | sub_agents Required | Count Constraint |
|------------|-------------------|------------------|
| `llm` | No | Any number (0+) |
| `parallel` | Yes | 2 or more |
| `loop` | Yes | Exactly 1 |
| `sequence` | Yes | 2 or more |

**Rules:**
- All referenced agent names must exist in team's `agents` section
- No circular references
- Agent names are case-sensitive

**Examples:**

**LLM agent (optional):**
```yaml
coordinator:
  type: llm
  sub_agents:
    - specialist_1
    - specialist_2
    - specialist_3
```

**Parallel agent (required, 2+):**
```yaml
multi_analyzer:
  type: parallel
  sub_agents:
    - perspective_1
    - perspective_2
    - perspective_3
```

**Loop agent (required, exactly 1):**
```yaml
refiner:
  type: loop
  max_iterations: 3
  sub_agents:
    - content_improver
```

**Sequence agent (required, 2+):**
```yaml
pipeline:
  type: sequence
  sub_agents:
    - stage_1
    - stage_2
    - stage_3
```

---

### toolsets

**Type:** `array` of strings  
**Required:** No  
**Supported by:** `llm` agent type only

Lists external tool integrations available to this agent.

**Rules:**
- Only LLM agents can have tools directly
- Parallel/Loop/Sequence agents cannot have tools (but their sub-agents can)
- All toolset names must be registered with system

**Common toolsets:**
- `duckduckgo` - Web search
- `memory-server` - Data persistence
- `notification-server` - Push notifications
- `weather-tool` - Weather data
- `google-calendar` - Calendar integration

**Examples:**

**Single tool:**
```yaml
weather_agent:
  type: llm
  toolsets:
    - weather-tool
```

**Multiple tools:**
```yaml
researcher:
  type: llm
  toolsets:
    - duckduckgo
    - memory-server
```

**Not supported:**
```yaml
parallel_agent:
  type: parallel
  toolsets:    # ❌ Error: parallel agents cannot have tools
    - duckduckgo
```

**Correct approach:**
```yaml
parallel_agent:
  type: parallel
  sub_agents:
    - researcher_1
    - researcher_2

researcher_1:
  type: llm
  toolsets:    # ✅ Correct: sub-agent has tools
    - duckduckgo
```

---

## Validation Rules

### Team-Level Validation

| Rule | Description |
|------|-------------|
| Required fields | `version`, `id`, `name`, `description`, `default_agent`, `agents` must be present |
| Unique ID | `id` must be unique across all teams |
| Default agent exists | `default_agent` must match an agent key in `agents` |
| At least one agent | `agents` must contain at least one agent |
| Unique setting keys | All setting `key` values must be unique |
| Setting types | `default_value` must match setting `type` |
| Allowed values | If present, `default_value` must be in `allowed_values` |

### Agent-Level Validation

| Rule | Description |
|------|-------------|
| Required fields | `name`, `description`, `instructions` must be present |
| Unique names | Agent names must be unique within team |
| Sub-agent references | All `sub_agents` must reference existing agents |
| Type constraints | Parallel (2+ sub-agents), Loop (1 sub-agent + max_iterations), Sequence (2+ sub-agents) |
| Max iterations range | Must be 1-10 for loop agents |
| Toolset availability | All `toolsets` must be registered |
| Tool restrictions | Only LLM agents can have `toolsets` |
| No circular refs | Agents cannot form circular delegation chains |

### Instruction Validation

| Rule | Description |
|------|-------------|
| Default required | Must have at least one instruction with `""` condition |
| Valid conditions | Conditions must be valid expressions |
| Setting references | Settings in conditions must exist in team `settings` |
| Non-empty content | Instruction text cannot be empty |

---

## Complete Example

```yaml
version: "1"
id: support-team
name: Customer Support Team
description: Routes customer support inquiries to specialized agents for technical, billing, and account assistance
interactive: true
default_agent: coordinator

settings:
  - key: priority_mode
    type: boolean
    label: Priority Mode
    description: Fast-track urgent requests
    default_value: false
  
  - key: response_style
    type: string
    label: Response Style
    description: Communication tone
    allowed_values:
      - professional
      - friendly
      - casual
    default_value: friendly

agents:
  coordinator:
    type: llm
    name: Support Coordinator
    description: Routes tickets to appropriate specialists
    model: gpt-4o
    instructions:
      - "settings.priority_mode": |
          PRIORITY MODE ACTIVE
          - Respond immediately
          - Escalate if needed
          - Follow up within 1 hour
      
      - |
          Analyze inquiry and route:
          - Technical issues → transfer_task to technical_support
          - Billing questions → transfer_task to billing_support
          - Account problems → transfer_task to account_support
    sub_agents:
      - technical_support
      - billing_support
      - account_support
    toolsets:
      - memory-server
  
  technical_support:
    type: llm
    name: Technical Support Specialist
    description: Handles technical issues and bugs
    model: gpt-4o
    instructions:
      - "settings.response_style == 'professional'": |
          Professional tone:
          - Formal language
          - "Thank you for contacting technical support"
          - "I will assist you with this technical issue"
      
      - "settings.response_style == 'friendly'": |
          Friendly tone:
          - Warm and approachable
          - "Hi! I'm happy to help with that technical issue"
          - "Let me help you get this resolved"
      
      - "settings.response_style == 'casual'": |
          Casual tone:
          - Relaxed and conversational
          - "Hey! Let's figure out what's going on"
          - "No worries, we'll get this fixed"
      
      - |
          Provide technical assistance:
          - Diagnose problems systematically
          - Offer step-by-step solutions
          - Link to documentation
          - Escalate complex issues
    toolsets:
      - memory-server
  
  billing_support:
    type: llm
    name: Billing Support Specialist
    description: Handles billing and payment inquiries
    instructions:
      - |
          Address billing questions:
          - Explain charges and invoices
          - Process refund requests
          - Update payment methods
          - Resolve billing disputes
          
          Always verify account ownership first.
    toolsets:
      - memory-server
  
  account_support:
    type: llm
    name: Account Support Specialist
    description: Handles account management and security
    instructions:
      - |
          Assist with account issues:
          - Reset passwords
          - Update profile information
          - Configure security settings
          - Handle account deletion requests
          
          Security-first: verify identity before changes.
    toolsets:
      - memory-server
```

---

## Next Steps

- **[Configuration Reference](../agents/configuration.md)** - Detailed property documentation
- **[Agent Examples](../agents/examples.md)** - Real-world templates
- **[Creating Teams](../agents/creating-agents.md)** - Step-by-step guide
- **[Settings](../advanced/settings.md)** - Runtime configuration
- **[Conditional Instructions](../advanced/conditional-instructions.md)** - Dynamic behavior

---

**Related:**  
[Configuration](../agents/configuration.md) | [Creating Teams](../agents/creating-agents.md) | [Agent Types](../agents/agent-types.md)
