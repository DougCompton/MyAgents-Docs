# Configuration Reference

Complete reference for all agent team configuration options.

## Team-Level Configuration

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

Unique identifier for the team across the system.

**Requirements:**
- Must start with lowercase letter
- Can contain: lowercase letters, numbers, hyphens
- Length: 3-50 characters
- Must be unique across all teams

**Examples:**
```yaml
# Valid
id: weather-team
id: customer-support
id: blog-writer-v2
id: analytics-dashboard

# Invalid
id: Weather-Team    # No uppercase
id: wt              # Too short (< 3)
id: _weather        # Cannot start with underscore
id: weather team    # No spaces
```

---

### name

**Type:** `string`  
**Required:** Yes  
**Length:** 5-100 characters

User-facing display name for the team.

```yaml
name: Weather Assistant Team
name: Customer Support
name: Blog Writing & Editing
```

**Guidelines:**
- Use title case
- Be descriptive and user-friendly
- Can include spaces and special characters
- Shown in UI and chat interface

---

### description

**Type:** `string`  
**Required:** Yes  
**Length:** 20-500 characters

Brief summary of team capabilities - **critical for switchboard routing decisions**.

```yaml
description: Provides current weather conditions, forecasts, and severe weather alerts for any location worldwide
```

**Best Practices:**
- ✅ Describe what problems the team solves
- ✅ Use action-oriented language
- ✅ Focus on user benefits
- ✅ Be specific about capabilities
- ❌ Avoid vague descriptions
- ❌ Don't describe implementation details

**Examples:**

**Good:**
```yaml
description: Routes customer support inquiries to specialized agents handling technical issues, billing questions, and account management

description: Researches topics using web search, creates comprehensive blog posts, and edits content for publication

description: Helps users find grocery products, plan meals, create shopping lists, and discover recipes based on dietary preferences
```

**Poor:**
```yaml
description: A team that helps with stuff

description: Uses multiple agents to process requests

description: Team for customer support
```

---

### default_agent

**Type:** `string`  
**Required:** Yes

Name of the entry-point agent - must match an agent name in the `agents` section.

```yaml
default_agent: coordinator

agents:
  coordinator:    # Must match default_agent
    # Configuration...
```

**Common Patterns:**
- `coordinator` - For teams with routing/delegation
- `assistant` - For single-purpose teams
- `manager` - For hierarchical teams
- Team-specific names (e.g., `shopping_assistant`, `weather_helper`)

---

### interactive

**Type:** `boolean`  
**Required:** No  
**Default:** `false`

Controls whether team remains active after processing messages.

```yaml
# Team persists for multi-turn conversations
interactive: true

# Team returns to switchboard after each response
interactive: false
```

**When to use `true`:**
- Shopping and browsing experiences
- Content creation with iteration
- Multi-step workflows
- Learning and tutorials

**When to use `false` (default):**
- Quick lookups
- Single-transaction tasks
- Automated processes
- Simple utilities

---

### settings

**Type:** `array` of setting objects  
**Required:** No

Defines runtime configuration options accessible to agents via conditional instructions.

```yaml
settings:
  - name: expert_mode
    type: bool
    title: Expert Mode
    description: Show advanced technical details
    defaultValue: false
  
  - name: temperature_unit
    type: string
    title: Temperature Unit
    description: Display temperatures in Fahrenheit or Celsius
    defaultValue: F
  
  - name: max_results
    type: number
    title: Maximum Results
    description: Number of search results to return
    defaultValue: 5
```

#### Setting Object Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Unique setting identifier (camelCase or snake_case) |
| `type` | string | Yes | Data type: `"boolean"`, `"string"`, or `"number"` |
| `title` | string | Yes | User-facing display name |
| `description` | string | Yes | Explanation of what the setting controls |
| `defaultValue` | boolean/string/number | Yes | Default value matching the type |

#### Setting Types

**Boolean Settings:**
```yaml
- name: verbose_mode
  type: bool
  title: Verbose Output
  description: Include detailed explanations and reasoning
  defaultValue: false
```

**String Settings:**
```yaml
- name: output_format
  type: string
  title: Output Format
  description: Format for generated content
  defaultValue: markdown
```

**Number Settings:**
```yaml
- name: result_limit
  type: number
  title: Result Limit
  description: Maximum number of items to return
  defaultValue: 10
```

See [Settings Guide](../advanced/settings.md) for detailed documentation.

---

### agents

**Type:** `object`  
**Required:** Yes

Map of agent names to agent configurations.

```yaml
agents:
  coordinator:
    # Agent configuration
  
  specialist_1:
    # Agent configuration
  
  specialist_2:
    # Agent configuration
```

**Requirements:**
- Must contain at least one agent
- Agent names must be unique within team
- `default_agent` must reference one of these agents

---

## Agent-Level Configuration

### type

**Type:** `string`  
**Required:** No  
**Default:** `"llm"`  
**Valid Values:** `"llm"`, `"parallel"`, `"loop"`, `"sequence"`

Specifies the agent execution pattern.

```yaml
# Standard interactive agent (default)
agent1:
  type: llm

# Concurrent execution
agent2:
  type: parallel

# Iterative refinement
agent3:
  type: loop
  max_iterations: 3

# Sequential pipeline
agent4:
  type: sequence
```

#### Type Comparison

| Type | Purpose | Requires sub_agents | Supports tools | Supports interactive |
|------|---------|---------------------|----------------|----------------------|
| `llm` | Interactive assistant | No | Yes | Yes |
| `parallel` | Concurrent execution | Yes | No* | No |
| `loop` | Iterative refinement | Yes (1 only) | No* | No |
| `sequence` | Sequential pipeline | Yes | No* | No |

\* Sub-agents can have tools

See [Agent Types](agent-types.md) for detailed documentation.

---

### name

**Type:** `string`  
**Required:** Yes  
**Pattern:** `^[a-z][a-z0-9_]{2,49}$`

Unique identifier for the agent within the team scope.

```yaml
agents:
  product_specialist:    # Valid
    name: Product Specialist
  
  price_analyst:         # Valid
    name: Price Analyst
```

**Requirements:**
- Must start with lowercase letter
- Can contain: lowercase letters, numbers, underscores
- Length: 3-50 characters
- Must be unique within team
- Use snake_case convention

**Examples:**
```yaml
# Valid
coordinator
technical_support
content_writer
data_analyst

# Invalid
ProductSpecialist    # No camelCase
ps                   # Too short
agent-1              # No hyphens (use underscores)
_coordinator         # Cannot start with underscore
```

---

### description

**Type:** `string`  
**Required:** Yes  
**Length:** 10-200 characters

Brief summary of the agent's role and responsibilities.

```yaml
agents:
  researcher:
    description: Conducts web research and gathers authoritative sources
  
  writer:
    description: Creates engaging, SEO-optimized blog content
  
  editor:
    description: Reviews and polishes content for grammar, clarity, and style
```

**Guidelines:**
- One sentence
- Focus on primary responsibility
- Use active voice
- Be specific about the agent's expertise

---

### model

**Type:** `string`  
**Required:** No  
**Valid Values:** `"anthropic"`, `"gpt-4o"`

Specifies which LLM provider to use for this agent.

```yaml
agents:
  analyst:
    model: anthropic    # Claude models
  
  writer:
    model: gpt-4o       # GPT-4 models
  
  calculator:
    # No model specified - uses system default
```

#### Model Selection Guide

**Claude (anthropic):**
- ✅ Long-form writing
- ✅ Detailed analysis
- ✅ Complex reasoning
- ✅ Code review
- ✅ Creative content

**GPT-4 (gpt-4o):**
- ✅ General-purpose tasks
- ✅ Broad knowledge
- ✅ Consistent performance
- ✅ Fast responses
- ✅ Multi-domain expertise

**When to specify:**
- Optimize for specific task types
- Manage costs (different pricing)
- Performance requirements
- Quality preferences

---

### instructions

**Type:** `array` of instruction objects  
**Required:** Yes

Defines agent behavior through conditional and default instructions.

#### Simple Instructions

```yaml
agents:
  assistant:
    instructions:
      - |
          You are a helpful assistant. Provide clear, accurate 
          responses to user questions.
```

#### Conditional Instructions

```yaml
agents:
  support_agent:
    instructions:
      - if: "settings.expert_mode"
        content: |
          You are in expert mode. Provide:
          - Technical details and internals
          - Advanced configuration options
          - API references and code examples
      
      - if: "settings.priority_mode"
        content: |
          This is a priority support request.
          - Respond immediately
          - Escalate if needed
          - Follow up within 1 hour
      
      - |
          You are a friendly support agent.
          - Use simple language
          - Provide step-by-step guidance
          - Confirm resolution before closing
```

#### Instruction Object Structure

Each instruction can be:
- **Plain string**: Default instruction (always included)
- **Conditional object**: Has `if` property (condition) and `content` property (instruction text)

#### Condition Syntax

**Boolean check (implicit truthiness):**
```yaml
- if: "settings.verbose_mode"
  content: |
    Instructions when verbose_mode is true
```

**Explicit boolean equality:**
```yaml
- if: "settings.expert_mode == true"
  content: |
    Instructions when expert mode is enabled

- if: "settings.expert_mode == false"
  content: |
    Instructions when expert mode is disabled
```

**String equality:**
```yaml
- if: "settings.format == \"json\""
  content: |
    Instructions for JSON format
```

**Note:** The schema does NOT support:
- ❌ Negation (`!settings.property`)
- ❌ Logical operators (`&&`, `||`)
- ❌ Comparison operators (`>`, `<`, `!=`)

Use separate conditional blocks for each condition instead.

See [Conditional Instructions](../advanced/conditional-instructions.md) for more patterns.

#### Best Practices

**Clear Role Definition:**
```yaml
instructions:
  - |
      You are a product specialist helping customers find the right products.
```

**Specific Responsibilities:**
```yaml
instructions:
  - |
      Your responsibilities:
      1. Search the product catalog
      2. Compare features and prices
      3. Provide 3-5 recommendations
      4. Answer product questions
```

**Business Rules:**
```yaml
instructions:
  - |
      Business rules:
      - Only recommend in-stock items
      - Highlight products with 4+ star ratings
      - Mention current sales and promotions
      - Disclose affiliate relationships
```

**Output Format:**
```yaml
instructions:
  - |
      Present recommendations in this format:
      
      **Product Name** - $Price
      - Key feature 1
      - Key feature 2
      - Rating: X/5 stars
```

**Tool Usage:**
```yaml
instructions:
  - |
      Tool usage:
      - Use duckduckgo for web research
      - Store user preferences in memory-server
      - Check weather-tool before making outdoor recommendations
```

**Delegation Logic:**
```yaml
instructions:
  - |
      Route requests as follows:
      - Technical issues → transfer_task to technical_support
      - Billing questions → transfer_task to billing_support
      - Account problems → transfer_task to account_support
```

---

### sub_agents

**Type:** `array` of strings  
**Required:** No  
**Required for:** `parallel`, `loop`, `sequence` agent types

Lists agents this agent can delegate to via `transfer_task` (for LLM agents) or execute (for special agent types).

```yaml
agents:
  coordinator:
    type: llm
    sub_agents:
      - researcher
      - writer
      - editor
  
  parallel_analyzer:
    type: parallel
    sub_agents:
      - perspective_1
      - perspective_2
      - perspective_3
  
  refinement_loop:
    type: loop
    max_iterations: 3
    sub_agents:
      - content_refiner    # Exactly 1 for loop type
  
  pipeline:
    type: sequence
    sub_agents:
      - stage_1
      - stage_2
      - stage_3
```

**Requirements by Type:**

| Type | sub_agents Required | Count Constraint |
|------|---------------------|------------------|
| `llm` | No | Any number |
| `parallel` | Yes | 2 or more |
| `loop` | Yes | Exactly 1 |
| `sequence` | Yes | 2 or more |

**Validation:**
- All referenced agent names must exist in team's `agents` section
- No circular references
- Agent names are case-sensitive

---

### toolsets

**Type:** `array` of strings  
**Required:** No  
**Supported by:** `llm` agent type only

Lists external tool integrations available to this agent.

```yaml
agents:
  researcher:
    type: llm
    toolsets:
      - duckduckgo        # Web search
      - memory-server     # Persistent storage
  
  scheduler:
    type: llm
    toolsets:
      - google-calendar   # Calendar integration
      - notification-server  # Push notifications
  
  coordinator:
    type: parallel
    # toolsets not supported for parallel agents
    # But sub-agents can have toolsets
```

#### Available Toolsets

**Common toolsets:**
- `duckduckgo` - Web search capabilities
- `memory-server` - Persistent key-value storage
- `notification-server` - Push notifications
- `weather-tool` - Weather data and forecasts
- `google-calendar` - Calendar events and scheduling

See [Available Tools](../tools/available-tools.md) for complete catalog.

#### Tool Access Rules

- ✅ LLM agents can use tools directly
- ❌ Parallel agents cannot (sub-agents can)
- ❌ Loop agents cannot (sub-agent can)
- ❌ Sequence agents cannot (sub-agents can)

---

### max_iterations

**Type:** `number`  
**Required:** Yes (for `loop` agents only)  
**Valid Range:** 1-10

Number of iterations for loop-type agents.

```yaml
agents:
  content_refiner:
    type: loop
    max_iterations: 3    # Will execute sub-agent 3 times
    sub_agents:
      - editor
```

**Guidelines:**
- 2-3 iterations: Most content refinement
- 4-5 iterations: Complex problem solving
- 6+ iterations: Rarely needed (diminishing returns)

**Cost implications:**
- Cost scales linearly with iterations
- Each iteration is a full agent execution
- Consider trade-off between quality and cost

---

## Complete Configuration Example

```yaml
version: "1"
id: research-writing-team
name: Research & Writing Team
description: Conducts comprehensive research and creates publication-ready content with professional editing
interactive: true
default_agent: project_manager

settings:
  - name: seo_mode
    type: bool
    title: SEO Optimization
    description: Optimize content for search engines
    defaultValue: false
  
  - name: tone
    type: string
    title: Content Tone
    description: Writing style and formality level
    defaultValue: professional
  
  - name: target_length
    type: number
    title: Target Word Count
    description: Desired article length in words
    defaultValue: 1500

agents:
  project_manager:
    type: llm
    name: Project Manager
    description: Coordinates research and writing workflow
    model: gpt-4o
    instructions:
      - |
          You coordinate the content creation process.
          
          Workflow:
          1. For research requests → transfer_task to multi_source_researcher
          2. For writing requests → transfer_task to content_pipeline
          3. Provide status updates to user
          
          Ensure quality at each stage.
    sub_agents:
      - multi_source_researcher
      - content_pipeline
  
  # Parallel agent for multi-source research
  multi_source_researcher:
    type: parallel
    name: Multi-Source Researcher
    description: Researches topic from multiple perspectives
    instructions:
      - |
          Synthesize research from all sources into a comprehensive brief:
          - Key facts and statistics
          - Different perspectives
          - Expert opinions
          - Source citations
    sub_agents:
      - academic_researcher
      - news_researcher
      - industry_researcher
  
  academic_researcher:
    type: llm
    name: Academic Researcher
    description: Finds scholarly sources and studies
    model: anthropic
    instructions:
      - |
          Focus on academic and scientific sources.
          Prioritize peer-reviewed research and authoritative institutions.
    toolsets:
      - duckduckgo
  
  news_researcher:
    type: llm
    name: News Researcher
    description: Finds current news and trends
    model: gpt-4o
    instructions:
      - |
          Focus on recent news articles and current events.
          Identify emerging trends and breaking developments.
    toolsets:
      - duckduckgo
  
  industry_researcher:
    type: llm
    name: Industry Researcher
    description: Finds industry reports and expert analysis
    model: anthropic
    instructions:
      - |
          Focus on industry publications, market research, and expert commentary.
          Identify practical applications and business implications.
    toolsets:
      - duckduckgo
  
  # Sequence agent for content creation pipeline
  content_pipeline:
    type: sequence
    name: Content Creation Pipeline
    description: Creates and refines content through structured workflow
    instructions:
      - |
          Process content through pipeline:
          1. Draft writer creates initial version
          2. SEO optimizer enhances discoverability (if enabled)
          3. Copy editor polishes final version
    sub_agents:
      - draft_writer
      - seo_optimizer
      - copy_editor
  
  draft_writer:
    type: llm
    name: Draft Writer
    description: Creates initial content draft
    model: anthropic
    instructions:
      - if: "settings.tone == 'professional'"
        content: |
          Write in a professional tone:
          - Formal language
          - Authoritative voice
          - Industry terminology
          - Data-driven arguments
      
      - if: "settings.tone == 'conversational'"
        content: |
          Write in a conversational tone:
          - Friendly, approachable language
          - Second-person perspective
          - Relatable examples
          - Engaging storytelling
      
      - if: "settings.tone == 'academic'"
        content: |
          Write in an academic tone:
          - Scholarly language
          - Evidence-based reasoning
          - Proper citations
          - Objective analysis
      
      - if: "settings.tone == 'casual'"
        content: |
          Write in a casual tone:
          - Relaxed, informal language
          - Personal anecdotes
          - Humor where appropriate
          - Easy-to-digest content
      
      - |
          Create comprehensive content based on research.
          Target length: {settings.target_length} words.
          
          Structure:
          - Compelling introduction
          - Well-organized body sections
          - Clear conclusion with takeaways
  
  seo_optimizer:
    type: llm
    name: SEO Optimizer
    description: Optimizes content for search engines
    model: gpt-4o
    instructions:
      - if: "settings.seo_mode"
        content: |
          Enhance content for SEO:
          - Identify target keyword
          - Optimize title and headings
          - Add meta description
          - Improve internal linking
          - Ensure keyword density (1-2%)
          - Add alt text recommendations
      
      - |
          Pass content through without SEO modifications.
  
  copy_editor:
    type: llm
    name: Copy Editor
    description: Final editing and polish
    model: anthropic
    instructions:
      - |
          Provide final polish:
          - Fix grammar and spelling
          - Improve clarity and flow
          - Strengthen weak sections
          - Ensure consistency
          - Verify tone matches requirements
          
          Maintain the author's voice while elevating quality.
```

## Validation Rules

### Team-Level Validation

| Rule | Description |
|------|-------------|
| Required fields | `version`, `id`, `name`, `description`, `default_agent`, `agents` must be present |
| Unique ID | `id` must be unique across all teams in system |
| Default agent exists | `default_agent` must match an agent name in `agents` |
| Agent count | Must have at least 1 agent |
| Setting keys unique | All setting `key` values must be unique within team |

### Agent-Level Validation

| Rule | Description |
|------|-------------|
| Required fields | `name`, `description`, `instructions` must be present |
| Unique names | Agent names must be unique within team |
| Sub-agent references | All `sub_agents` must reference existing agents in team |
| Type constraints | `parallel` needs 2+ sub-agents, `loop` needs exactly 1, `sequence` needs 2+ |
| Max iterations | Required for `loop` type, must be 1-10 |
| Toolset availability | All `toolsets` must be registered with system |
| No circular refs | Agents cannot form circular delegation chains |

### Instruction Validation

| Rule | Description |
|------|-------------|
| Default required | Must have at least one plain string instruction (no `if` condition) |
| Valid conditions | Conditions must match pattern `settings.property` or `settings.property == value` |
| Setting references | Settings referenced in conditions must exist in team `settings` |
| Non-empty | Instruction content cannot be empty |

### Setting Validation

| Rule | Description |
|------|-------------|
| Type match | `default_value` must match `type` |
| Allowed values | If `allowed_values` specified, `default_value` must be in list |
| String values only | `allowed_values` only valid for `type: string` |
| Key format | `key` must be snake_case, alphanumeric with underscores |

## Schema Errors

Common validation errors and how to fix them:

### "Agent 'X' not found"

```yaml
# ❌ Problem
default_agent: coordinater  # Typo

agents:
  coordinator:    # Actual name
    # ...

# ✅ Solution
default_agent: coordinator
```

### "Circular agent reference detected"

```yaml
# ❌ Problem
agents:
  agent_a:
    sub_agents:
      - agent_b
  
  agent_b:
    sub_agents:
      - agent_a    # Circular!

# ✅ Solution - Remove circular reference
agents:
  agent_a:
    sub_agents:
      - agent_b
  
  agent_b:
    # No sub_agents or delegate elsewhere
```

### "Loop agent requires exactly 1 sub-agent"

```yaml
# ❌ Problem
refiner:
  type: loop
  max_iterations: 3
  sub_agents:
    - agent_a
    - agent_b    # Too many!

# ✅ Solution
refiner:
  type: loop
  max_iterations: 3
  sub_agents:
    - agent_a    # Exactly 1
```

### "Toolset 'X' not available"

```yaml
# ❌ Problem
researcher:
  toolsets:
    - duck-duck-go    # Wrong name

# ✅ Solution
researcher:
  toolsets:
    - duckduckgo      # Correct name
```

### "Default value must be in allowed_values"

```yaml
# ❌ Problem
settings:
  - name: format
    type: string
    defaultValue: yaml    # Value should be from a known set

# ✅ Solution
settings:
  - name: format
    type: string
    description: Output format (json, xml, or yaml)
    defaultValue: yaml
```

## Next Steps

- **[Creating Agents](creating-agents.md)** - Step-by-step guide
- **[Agent Types](agent-types.md)** - LLM, Parallel, Loop, Sequence details
- **[Examples](examples.md)** - Real-world configuration examples
- **[Settings Guide](../advanced/settings.md)** - Advanced settings patterns
- **[Conditional Instructions](../advanced/conditional-instructions.md)** - Dynamic behavior

---

**Related:**  
[Overview](overview.md) | [Creating Teams](creating-agents.md) | [Schema Reference](../reference/schema.md)
