# Creating Agent Teams

This guide walks you through creating your first AI agent team from scratch.

## Prerequisites

Before creating teams, ensure you have:

- ✅ Access to the MyAgents API
- ✅ Understanding of [Team Architecture](overview.md)
- ✅ Familiarity with [Agent Types](agent-types.md)
- ✅ Basic YAML knowledge

## Quick Start

The fastest way to create a team:

1. **Create YAML file** in `apps/api/src/agents/`
2. **Define team identity** (id, name, description)
3. **Add agents** with instructions
4. **Restart API** to load team
5. **Test** via chat interface

## Step-by-Step Example

Create `apps/api/src/agents/weather-team.yaml`:

```yaml
version: "1"
id: weather-team
name: Weather Team
description: Provides current weather and forecasts
default_agent: weather_helper

agents:
  weather_helper:
    type: llm
    name: Weather Helper
    description: Retrieves weather information
    instructions:
      - |
          You provide weather information using the weather tool.
          Present forecasts clearly with temperatures, conditions, and alerts.
    toolsets:
      - weather-tool
```

Restart the API, and your team is ready!

## Step 1: Team Identity

Every team needs a unique identity for routing and discovery.

### Required Fields

#### `version` (string)
Schema version for future compatibility.

```yaml
version: "1"  # Always use "1" currently
```

#### `id` (string)
Unique system identifier for this team.

**Requirements:**
- Lowercase letters, numbers, hyphens only
- Start with letter
- Between 3-50 characters
- Unique across all teams

**Examples:**
```yaml
# Good
id: weather-team
id: customer-support
id: content-creator-v2

# Bad
id: Weather Team  # No spaces
id: wt            # Too short
id: _weather      # No leading underscore
```

#### `name` (string)
User-facing display name.

**Guidelines:**
- Clear and descriptive
- Title case recommended
- 5-50 characters
- Can include spaces and special characters

**Examples:**
```yaml
# Good
name: Weather Assistant
name: Customer Support Team
name: Blog Writing & Editing

# Bad
name: wt              # Not descriptive
name: Team            # Too generic
```

#### `description` (string)
Brief summary of team capabilities - **critical for switchboard routing**.

**Guidelines:**
- One sentence, 50-200 characters
- Describe what problems the team solves
- Focus on user benefits, not internal implementation
- Use active, specific language

**Examples:**
```yaml
# Good - Specific and action-oriented
description: Provides current weather conditions and multi-day forecasts for any location

description: Routes customer support inquiries to specialized agents for technical, billing, or account issues

description: Researches, writes, and edits blog posts with SEO optimization

# Bad - Vague or implementation-focused
description: A team that does weather stuff

description: Uses agents to help users

description: Has multiple personas for different tasks
```

#### `default_agent` (string)
Entry point agent name - first agent that handles requests.

```yaml
default_agent: coordinator  # Must match an agent name in agents section
```

### Optional Fields

#### `interactive` (boolean)
Controls whether team persists after processing requests.

```yaml
# Team stays active for multi-turn conversations
interactive: true

# Team returns to switchboard after each response (default)
interactive: false
```

**When to use `interactive: true`:**
- Shopping and browsing experiences
- Content creation and iteration
- Multi-step workflows requiring context
- Tutorial and learning experiences

**When to use `interactive: false` (default):**
- Quick lookups (weather, definitions, calculations)
- Single-transaction tasks
- Automated notifications
- Background processing

#### `settings` (object)
Runtime configuration options accessible to agents.

```yaml
settings:
  - name: verbose_mode
    type: boolean
    title: Verbose Output
    description: Include detailed explanations
    defaultValue: false
  
  - name: temperature_unit
    type: string
    title: Temperature Unit
    description: Display temperature in Fahrenheit or Celsius
    defaultValue: F
```

See [Settings](../advanced/settings.md) for complete documentation.

### Complete Team Header

```yaml
version: "1"
id: support-team
name: Customer Support Team
description: Routes customer inquiries to specialized support agents for technical, billing, and account assistance
interactive: true
default_agent: coordinator

settings:
  - name: priority_mode
    type: boolean
    title: Priority Mode
    description: Fast-track urgent requests
    defaultValue: false

agents:
  # Agents defined below...
```

## Step 2: Define Agents

Agents are the specialized roles that make up your team.

### Single-Agent Team

The simplest team has one agent handling all interactions.

```yaml
version: "1"
id: calculator-team
name: Calculator Team
description: Performs mathematical calculations
default_agent: calculator

agents:
  calculator:
    type: llm
    name: Calculator
    description: Performs calculations
    instructions:
      - |
          You perform mathematical calculations. Handle:
          - Basic arithmetic
          - Unit conversions
          - Percentage calculations
          - Statistical operations
          
          Show your work and provide clear explanations.
```

**Best for:**
- Specialized utilities
- Single-purpose tools
- Simple workflows

### Multi-Agent Team

Complex teams have multiple agents that coordinate.

```yaml
version: "1"
id: shopping-team
name: Shopping Team
description: Assists with grocery shopping, meal planning, and product recommendations
interactive: true
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Shopping Coordinator
    description: Routes shopping requests
    instructions:
      - |
          Route customer requests:
          - Product searches → product_specialist
          - Meal planning → meal_planner
          - Price comparisons → price_analyst
          
          Use transfer_task to delegate.
    sub_agents:
      - product_specialist
      - meal_planner
      - price_analyst
  
  product_specialist:
    type: llm
    name: Product Specialist
    description: Finds and recommends products
    instructions:
      - |
          Search catalog for products matching customer needs.
          Provide 3-5 recommendations with:
          - Product name and description
          - Price and size
          - Rating and review summary
          - Availability
    toolsets:
      - product-catalog
  
  meal_planner:
    type: llm
    name: Meal Planner
    description: Creates meal plans and shopping lists
    instructions:
      - |
          Create meal plans based on:
          - Dietary restrictions
          - Number of servings
          - Budget constraints
          - Cuisine preferences
          
          Generate shopping lists with quantities.
  
  price_analyst:
    type: llm
    name: Price Analyst
    description: Compares prices and finds deals
    instructions:
      - |
          Compare prices across brands and sizes.
          Identify:
          - Best value (price per unit)
          - Current sales and discounts
          - Bulk purchase savings
          - Generic alternatives
    toolsets:
      - product-catalog
```

## Step 3: Agent Configuration

Each agent requires specific configuration properties.

### Required Properties

#### `name` (string)
Unique identifier within team scope.

**Naming conventions:**
```yaml
# Good - Descriptive, snake_case
name: product_specialist
name: technical_support
name: content_writer

# Bad
name: ps              # Too short
name: agent1          # Not descriptive
name: Product-Agent   # Use snake_case
```

#### `description` (string)
Brief summary of agent role (50-150 characters).

```yaml
# Good
description: Finds products matching customer requirements
description: Handles technical troubleshooting and bug reports
description: Creates engaging blog content with SEO optimization

# Bad
description: Agent    # Not descriptive
description: Does product stuff  # Too vague
```

#### `instructions` (array)
Defines agent behavior - the most important configuration.

**Structure:**
```yaml
instructions:
  - "condition": |
      Instructions when condition is true
  - |
      Default instructions (condition is empty string)
```

**Basic Example:**
```yaml
instructions:
  - |
      You are a technical support agent.
      
      Responsibilities:
      - Diagnose technical issues
      - Provide step-by-step solutions
      - Escalate complex problems
      
      Guidelines:
      - Be patient and clear
      - Confirm understanding before closing
      - Document all resolutions
```

**With Conditional Logic:**
```yaml
instructions:
  - "settings.expert_mode": |
      You are in expert mode. Provide:
      - Technical details and internals
      - Advanced configuration options
      - Performance optimization tips
      - Direct API access when relevant
  - |
      You are in standard mode. Provide:
      - User-friendly explanations
      - Step-by-step guides
      - Visual aids when helpful
      - No technical jargon
```

See [Conditional Instructions](../advanced/conditional-instructions.md) for advanced patterns.

### Optional Properties

#### `type` (string)
Execution pattern for this agent.

```yaml
type: llm        # Interactive assistant (default)
type: parallel   # Concurrent execution
type: loop       # Iterative refinement
type: sequence   # Sequential pipeline
```

See [Agent Types](agent-types.md) for detailed documentation.

#### `model` (string)
LLM provider for this agent.

```yaml
model: anthropic  # Claude (recommended for analysis, writing)
model: gpt-4o     # GPT-4 (general purpose)
```

**When to specify:**
- Cost optimization (different pricing)
- Quality requirements (specific model strengths)
- Performance needs (response time variations)

**Model Strengths:**
- **Claude (anthropic)**: Long-form writing, detailed analysis, complex reasoning
- **GPT-4 (gpt-4o)**: General tasks, broad knowledge, consistent performance

#### `sub_agents` (array of strings)
Agents this agent can delegate to via `transfer_task`.

```yaml
coordinator:
  sub_agents:
    - specialist_1
    - specialist_2
    - specialist_3
```

**Important:**
- Agent names must exist in team's `agents` section
- Creates delegation hierarchy
- Enables workflow orchestration

#### `toolsets` (array of strings)
External tools this agent can access.

```yaml
researcher:
  toolsets:
    - duckduckgo      # Web search
    - memory-server   # Persistent storage
```

See [Available Tools](../tools/available-tools.md) for complete catalog.

### Complete Agent Example

```yaml
content_writer:
  type: llm
  name: Content Writer
  description: Creates SEO-optimized blog posts
  model: anthropic
  instructions:
    - "settings.seo_mode": |
        Create SEO-optimized content with:
        - Target keyword integration (5-7 mentions)
        - Meta description under 160 characters
        - Header hierarchy (H2, H3)
        - Internal linking opportunities
        - Alt text for images
    - |
        Create engaging blog content with:
        - Clear introduction with hook
        - Structured body with subheadings
        - Actionable takeaways
        - Compelling conclusion
        
        Tone: Professional yet conversational
        Length: 1000-1500 words
  toolsets:
    - duckduckgo
    - memory-server
  sub_agents:
    - seo_analyst
    - fact_checker
```

## Step 4: Agent Coordination

### Delegation Patterns

#### Hub-and-Spoke (Most Common)
One coordinator routes to multiple specialists.

```
    Coordinator
    /    |    \
   /     |     \
  A      B      C
```

```yaml
agents:
  coordinator:
    sub_agents:
      - specialist_a
      - specialist_b
      - specialist_c
  
  specialist_a:
    # Configuration...
  
  specialist_b:
    # Configuration...
  
  specialist_c:
    # Configuration...
```

#### Hierarchical
Multi-level delegation for complex workflows.

```
   Manager
      |
   Team Lead
    /    \
   A      B
```

```yaml
agents:
  manager:
    sub_agents:
      - team_lead
  
  team_lead:
    sub_agents:
      - specialist_a
      - specialist_b
  
  specialist_a:
    # Configuration...
  
  specialist_b:
    # Configuration...
```

#### Sequential Pipeline
Use `sequence` agent type for ordered processing.

```
A → B → C → D
```

```yaml
agents:
  pipeline:
    type: sequence
    sub_agents:
      - stage_a
      - stage_b
      - stage_c
      - stage_d
```

See [Agent Types](agent-types.md) and [Multi-Agent Workflows](../advanced/multi-agent.md).

### Transfer Task Pattern

Agents use the automatically available `transfer_task` function to delegate:

```yaml
coordinator:
  instructions:
    - |
        Analyze the request type:
        
        - Technical questions: use transfer_task to delegate to technical_support
        - Billing questions: use transfer_task to delegate to billing_support
        - Account questions: use transfer_task to delegate to account_support
        
        Include relevant context when transferring.
  sub_agents:
    - technical_support
    - billing_support
    - account_support
```

**Key points:**
- `transfer_task` is automatically available to agents with `sub_agents`
- Agent names must match `sub_agents` array
- Include context in transfer for continuity

## Step 5: Add Tools

Tools connect agents to external systems and data.

### Available Toolsets

Common toolsets include:
- `duckduckgo` - Web search
- `memory-server` - Persistent storage
- `notification-server` - Push notifications
- `weather-tool` - Weather data
- `google-calendar` - Calendar integration

See [Available Tools](../tools/available-tools.md) for complete list.

### Tool Configuration

Add toolsets to agent configuration:

```yaml
researcher:
  name: Researcher
  description: Conducts web research
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Conduct thorough research using web search.
        Store important findings in memory for future reference.
```

### Tool Usage in Instructions

Guide agents on when and how to use tools:

```yaml
shopping_assistant:
  toolsets:
    - product-catalog
    - memory-server
  instructions:
    - |
        When user asks about products:
        1. Check memory-server for user preferences
        2. Search product-catalog using preferences
        3. Present 3-5 recommendations
        4. Store user feedback in memory-server
```

### Tool Best Practices

1. **Minimal Tools**: Only include tools agent actually needs
2. **Clear Instructions**: Specify when to use each tool
3. **Error Handling**: Guide agents on tool failures
4. **Data Management**: Define what to store/retrieve from memory

## Step 6: Testing & Validation

### Local Testing

1. **Place file** in `apps/api/src/agents/`
2. **Restart API** to load team
3. **Check logs** for loading errors
4. **Test via chat** interface

### Validation Checklist

**Team Configuration:**
- ✅ Unique `id` (no conflicts)
- ✅ Descriptive `name` and `description`
- ✅ `default_agent` matches an agent name
- ✅ Valid YAML syntax

**Agent Configuration:**
- ✅ All agents have unique names
- ✅ All agents have `description`
- ✅ All agents have `instructions`
- ✅ `sub_agents` reference valid agent names
- ✅ `toolsets` reference available tools

**Functionality:**
- ✅ Default agent responds appropriately
- ✅ Delegation works as expected
- ✅ Tools are accessible and functional
- ✅ Settings apply correctly (if used)

### Common Issues

**Team Not Loading**
```
Error: Duplicate team ID
```
- ✅ Check: Is `id` unique across all teams?
- ✅ Fix: Choose different `id`

**YAML Syntax Error**
```
Error: Invalid YAML at line 42
```
- ✅ Check: Proper indentation (use spaces, not tabs)
- ✅ Check: Strings with colons in quotes
- ✅ Check: Multiline strings use `|` correctly

**Agent Not Found**
```
Error: Default agent 'coordinater' not found
```
- ✅ Check: Spelling matches exactly (case-sensitive)
- ✅ Check: Agent exists in `agents` section

**Tool Not Available**
```
Error: Toolset 'duck-duck-go' not found
```
- ✅ Check: Correct toolset name (e.g., `duckduckgo`)
- ✅ Check: Toolset is registered with system

### Debugging Tips

1. **Check API Logs**: Look for loading and validation errors
2. **Test Incrementally**: Start simple, add complexity gradually
3. **Verify Syntax**: Use YAML validator before deploying
4. **Review Examples**: Compare with working team configurations

## Step 7: Deployment

### Development Environment

1. Place YAML file in `apps/api/src/agents/`
2. Restart API server
3. Team loads automatically

### Production Deployment

1. **Validate** thoroughly in development
2. **Test** all functionality
3. **Document** team capabilities
4. **Deploy** via standard CI/CD pipeline
5. **Monitor** usage and performance

### Version Control

```bash
# Add team file to git
git add apps/api/src/agents/my-team.yaml

# Commit with descriptive message
git commit -m "Add shopping team with meal planning"

# Push to repository
git push origin main
```

## Examples by Use Case

### Customer Support Team

```yaml
version: "1"
id: support-team
name: Customer Support
description: Routes support tickets to technical, billing, or account specialists
interactive: true
default_agent: coordinator

agents:
  coordinator:
    name: Support Coordinator
    description: Routes tickets to specialists
    instructions:
      - |
          Analyze the inquiry and delegate:
          - Bugs, errors, technical issues → technical_support
          - Payments, invoices, billing → billing_support
          - Login, settings, security → account_support
    sub_agents:
      - technical_support
      - billing_support
      - account_support
  
  technical_support:
    name: Technical Support
    description: Handles technical issues
    instructions:
      - |
          Provide technical assistance:
          - Diagnose problems systematically
          - Offer step-by-step solutions
          - Link to relevant documentation
          - Escalate complex issues to engineering
    toolsets:
      - memory-server
  
  billing_support:
    name: Billing Support
    description: Handles billing inquiries
    instructions:
      - |
          Address billing questions:
          - Explain charges and invoices
          - Process refund requests
          - Update payment methods
          - Resolve billing disputes
    toolsets:
      - memory-server
  
  account_support:
    name: Account Support
    description: Handles account management
    instructions:
      - |
          Assist with account issues:
          - Reset passwords
          - Update profile information
          - Configure security settings
          - Delete or deactivate accounts
    toolsets:
      - memory-server
```

### Research & Writing Team

```yaml
version: "1"
id: research-writing-team
name: Research & Writing Team
description: Conducts research and creates comprehensive written content
interactive: true
default_agent: coordinator

agents:
  coordinator:
    name: Project Coordinator
    description: Manages research and writing workflow
    instructions:
      - |
          Coordinate content creation:
          1. Route research requests to researcher
          2. Route writing requests to writer with research
          3. Route editing requests to editor
          
          Ensure quality throughout pipeline.
    sub_agents:
      - researcher
      - writer
      - editor
  
  researcher:
    name: Researcher
    description: Conducts thorough research
    model: anthropic
    instructions:
      - |
          Research the topic comprehensively:
          - Find authoritative sources
          - Gather key facts and statistics
          - Identify different perspectives
          - Note conflicting information
          
          Provide organized research brief.
    toolsets:
      - duckduckgo
      - memory-server
  
  writer:
    name: Content Writer
    description: Creates written content
    model: anthropic
    instructions:
      - |
          Write engaging content based on research:
          - Clear, compelling introduction
          - Well-structured body with subheadings
          - Evidence-based arguments
          - Strong conclusion
          
          Tone: Professional yet accessible
          Length: 1000-1500 words
  
  editor:
    name: Editor
    description: Polishes content
    model: anthropic
    instructions:
      - |
          Edit for excellence:
          - Fix grammar and spelling
          - Improve clarity and flow
          - Strengthen arguments
          - Ensure consistency
          
          Maintain author's voice while elevating quality.
```

### Data Analysis Team

```yaml
version: "1"
id: analytics-team
name: Analytics Team
description: Analyzes data and generates insights with visualizations
default_agent: analyzer

agents:
  analyzer:
    type: sequence
    name: Analysis Pipeline
    description: Complete analysis workflow
    instructions:
      - |
          Process data through pipeline:
          1. Validate and clean data
          2. Perform statistical analysis
          3. Generate summary report
    sub_agents:
      - data_validator
      - statistician
      - report_writer
  
  data_validator:
    name: Data Validator
    description: Validates and cleans data
    instructions:
      - |
          Validate the dataset:
          - Check for missing values
          - Identify outliers
          - Verify data types
          - Remove duplicates
          
          Report data quality issues.
  
  statistician:
    name: Statistician
    description: Performs statistical analysis
    model: gpt-4o
    instructions:
      - |
          Analyze the data:
          - Calculate descriptive statistics
          - Identify trends and patterns
          - Test hypotheses
          - Determine correlations
          
          Provide clear, actionable insights.
  
  report_writer:
    name: Report Writer
    description: Creates analysis reports
    model: anthropic
    instructions:
      - |
          Generate comprehensive report:
          - Executive summary
          - Key findings
          - Detailed analysis
          - Recommendations
          - Methodology notes
          
          Use clear visualizations and charts.
```

## Best Practices

### Team Design

1. **Single Responsibility**: Each team should have a clear, focused purpose
2. **Clear Boundaries**: Define what the team does and doesn't handle
3. **Appropriate Scope**: Not too broad, not too narrow
4. **User-Focused**: Design around user needs, not implementation details

### Agent Design

1. **Specialized Roles**: Each agent should have specific expertise
2. **Clear Instructions**: Detailed, actionable behavioral guidelines
3. **Appropriate Tools**: Only tools the agent actually needs
4. **Delegation Logic**: When and why to delegate to other agents

### Configuration

1. **Descriptive Names**: Use clear, consistent naming
2. **Helpful Descriptions**: Aid routing and user understanding
3. **Sensible Defaults**: Settings should work well out-of-the-box
4. **Validation**: Test thoroughly before deployment

### Maintenance

1. **Version Control**: Track all changes
2. **Documentation**: Explain team purpose and capabilities
3. **Monitoring**: Track usage and performance
4. **Iteration**: Refine based on user feedback

## Next Steps

- **[Configuration Reference](configuration.md)** - Complete schema documentation
- **[Agent Types](agent-types.md)** - LLM, Parallel, Loop, Sequence details
- **[Examples](examples.md)** - More real-world templates
- **[Tools](../tools/overview.md)** - Learn about available tools
- **[Advanced Patterns](../advanced/multi-agent.md)** - Complex workflows

---

**Related:**  
[Overview](overview.md) | [Agent Types](agent-types.md) | [Configuration](configuration.md)
