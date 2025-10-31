# Conditional Instructions

Dynamic agent behavior using conditional logic based on settings.

## What are Conditional Instructions?

**Conditional instructions** allow agents to adapt their behavior based on:

- ⚙️ **Settings** - User preferences and configuration
- 🔄 **Dynamic behavior** - Different instructions for different scenarios
- 🎛️ **Feature flags** - Enable/disable functionality

## Basic Syntax

### Instruction Structure

```yaml
instructions:
  - if: "settings.setting_name == value"
    content: |
      Instructions when condition is true
  - |
    Default instructions (always included)
```

### Simple Example

```yaml
agents:
  assistant:
    instructions:
      - if: "settings.expert_mode == true"
        content: |
          You are in expert mode. Provide technical details,
          use industry terminology, and include advanced concepts.
      
      - |
        You are in standard mode. Use accessible language
        and explain concepts clearly.
```

## Supported Conditions

### Boolean Settings

**Check if boolean setting is true:**
```yaml
instructions:
  - if: "settings.verbose == true"
    content: |
      Verbose mode active - provide detailed explanations
```

**Check if boolean setting is false:**
```yaml
instructions:
  - if: "settings.verbose == false"
    content: |
      Concise mode - keep responses brief
```

**Note:** The `content` property holds the instruction text when using `if` conditions.

### String Settings

**String equality:**
```yaml
instructions:
  - if: "settings.format == \"json\""
    content: |
      Output in JSON format
```

**String inequality:**
```yaml
instructions:
  - if: "settings.dietary_restrictions != \"none\""
    content: |
      Filter recipes by restriction: {settings.dietary_restrictions}
```

**Note:** String values must be double-quoted within the condition.

### Supported Operators

**✅ Fully Supported:**
- `==` - Equality (booleans, strings, numbers)
- `!=` - Inequality (booleans, strings, numbers)

**❌ NOT Supported:**
- Negation operators (`!`)
- Logical operators (`&&`, `||`)
- Comparison operators (`>`, `<`, `>=`, `<=`)
- Complex expressions with parentheses

**Workaround for logical operators:** Use separate explicit conditions for each case:

```yaml
instructions:
  - if: "settings.verbose == true"
    content: |
      Verbose instructions
  
  - |
    Default (non-verbose) instructions include implicit handling
    of when verbose is false
```

## Evaluation Order

Instructions are evaluated **in order** from top to bottom. All matching conditions are included and concatenated.

### Best Practice Ordering

```yaml
# ✅ GOOD - Base instructions first, then conditionals
instructions:
  - |
    Default instructions (base behavior)
  
  - if: "settings.expert_mode == true"
    content: |
      Additional expert instructions
```

## Common Patterns

### Mode Switching

```yaml
settings:
  - name: mode
    type: string
    title: Experience Mode
    description: Adjust explanations based on user experience level
    defaultValue: intermediate

agents:
  instructor:
    instructions:
      - if: "settings.mode == \"beginner\""
        content: |
          BEGINNER MODE
          - Define all terms
          - Start with basics
          - Step-by-step guidance
          - Lots of examples
          - Avoid jargon
      
      - if: "settings.mode == \"intermediate\""
        content: |
          INTERMEDIATE MODE
          - Assume basic knowledge
          - Practical focus
          - Some technical terms okay
          - Balanced explanations
      
      - if: "settings.mode == \"expert\""
        content: |
          EXPERT MODE
          - Advanced concepts
          - Technical terminology
          - Architectural discussions
          - Best practices and trade-offs
      
      - |
        Standard instruction set
```

### Feature Flags

```yaml
settings:
  - name: enable_web_search
    type: bool
    title: Enable Web Search
    description: Allow searching the web for current information
    defaultValue: true

  - name: enable_memory
    type: bool
    title: Enable Memory
    description: Store and retrieve context from memory
    defaultValue: true

agents:
  assistant:
    toolsets:
      - duckduckgo
      - memory-server
    instructions:
      - |
        Base capabilities:
        - Answer questions using training data
      
      - if: "settings.enable_web_search == true"
        content: |
          Web search enabled:
          - Use web search for current information
          - Include sources in responses
      
      - if: "settings.enable_memory == true"
        content: |
          Memory enabled:
          - Store important findings for future reference
          - Retrieve relevant context from previous interactions
          - Build knowledge base over time
```

### Output Format Switching

```yaml
settings:
  - name: output_format
    type: string
    title: Output Format
    description: Format for generated content
    defaultValue: markdown

agents:
  formatter:
    instructions:
      - if: "settings.output_format == \"markdown\""
        content: |
          Format in Markdown:
          - Use # for headings
          - Use **bold** and *italic*
          - Use - for bullet points
          - Use ``` for code blocks
      
      - if: "settings.output_format == \"json\""
        content: |
          Format as JSON:
          - Valid JSON syntax
          - Proper escaping
          - No trailing commas
          - Include structure: {"title": "...", "content": "..."}
      
      - if: "settings.output_format == \"html\""
        content: |
          Format as HTML:
          - Use semantic tags
          - Proper nesting
          - Close all tags
      
      - |
        Use markdown format (default)
```

### Tone Adaptation

```yaml
settings:
  - name: tone
    type: string
    title: Content Tone
    description: Writing style and formality
    defaultValue: friendly

agents:
  customer_service:
    instructions:
      - if: "settings.tone == \"professional\""
        content: |
          Professional tone:
          - "Thank you for contacting us"
          - "I'd be happy to assist you with..."
          - Use proper grammar and formal language
      
      - if: "settings.tone == \"friendly\""
        content: |
          Friendly tone:
          - "Hi there! How can I help?"
          - "I'm happy to help you with that!"
          - Warm and approachable
      
      - if: "settings.tone == \"casual\""
        content: |
          Casual tone:
          - "Hey! What's up?"
          - "Sure thing! Let me help you out"
          - Relaxed and conversational
      
      - |
        Standard professional-friendly tone
```

### Priority Levels

```yaml
settings:
  - name: priority
    type: string
    title: Priority Level
    description: Request priority level
    defaultValue: normal

agents:
  support_agent:
    toolsets:
      - notification-server
    instructions:
      - if: "settings.priority == \"urgent\""
        content: |
          🚨 URGENT PRIORITY
          - Respond immediately
          - Escalate to senior team if needed
          - Send notification to managers
          - Follow up within 15 minutes
      
      - if: "settings.priority == \"high\""
        content: |
          ⚡ HIGH PRIORITY
          - Prioritize this request
          - Respond within 1 hour
          - Notify relevant team members
      
      - if: "settings.priority == \"normal\""
        content: |
          📋 NORMAL PRIORITY
          - Standard response time (within 4 hours)
          - Follow normal procedures
      
      - if: "settings.priority == \"low\""
        content: |
          📌 LOW PRIORITY
          - Process when time allows
          - Standard response within 24 hours
      
      - |
          Standard support workflow
```

## Advanced Patterns

### Layered Feature Enablement

Build up capabilities with multiple feature flags:

```yaml
settings:
  - name: enable_search
    type: bool
    title: Enable Search
    description: Enable web search capabilities
    defaultValue: true

  - name: enable_memory
    type: bool
    title: Enable Memory
    description: Enable persistent memory
    defaultValue: true

  - name: enable_notifications
    type: bool
    title: Enable Notifications
    description: Enable push notifications
    defaultValue: false

agents:
  assistant:
    toolsets:
      - duckduckgo
      - memory-server
      - notification-server
    instructions:
      - |
          Core capabilities:
          - Answer questions using training data
      
      - if: "settings.enable_search == true"
        content: |
          Search capability enabled:
          - Use web search for current information
          - Cite sources
      
      - if: "settings.enable_memory == true"
        content: |
          Memory capability enabled:
          - Store important context
          - Retrieve relevant history
          - Build knowledge over time
      
      - if: "settings.enable_notifications == true"
        content: |
          Notification capability enabled:
          - Send notifications for important updates
          - Alert user of completed tasks
```

### Context-Aware Behavior

Adjust behavior based on user type:

```yaml
settings:
  - name: user_type
    type: string
    title: User Type
    description: Type of user (student, professional, researcher)
    defaultValue: professional
  
  - name: detail_level
    type: string
    title: Detail Level
    description: Level of detail in responses (brief, normal, detailed)
    defaultValue: normal

agents:
  adaptive_assistant:
    instructions:
      - |
          Base instructions for all users
      
      - if: "settings.user_type == \"student\""
        content: |
          Student context:
          - Educational focus
          - Define technical terms
          - Include learning resources
      
      - if: "settings.user_type == \"professional\""
        content: |
          Professional context:
          - Business implications
          - Practical applications
          - ROI considerations
      
      - if: "settings.user_type == \"researcher\""
        content: |
          Researcher context:
          - Academic rigor
          - Methodology details
          - Citations and references
      
      - if: "settings.detail_level == \"brief\""
        content: |
          Brief mode:
          - Concise responses
          - Key points only
          - Executive summary style
      
      - if: "settings.detail_level == \"detailed\""
        content: |
          Detailed mode:
          - Comprehensive explanations
          - In-depth analysis
          - Multiple examples
```

### Conditional Tool Access

Control which tools are used based on settings:

```yaml
settings:
  - name: data_source
    type: string
    title: Data Source
    description: Where to retrieve information from
    defaultValue: web_and_memory

agents:
  information_agent:
    toolsets:
      - duckduckgo
      - memory-server
    instructions:
      - |
          Information retrieval agent
      
      - if: "settings.data_source == \"web_only\""
        content: |
          Web-only mode:
          - Use duckduckgo for all queries
          - Do not use memory-server
          - Fresh data every time
      
      - if: "settings.data_source == \"memory_only\""
        content: |
          Memory-only mode:
          - Search memory-server only
          - Do not use web search
          - Return cached data if available
          - Inform user if data not in memory
      
      - if: "settings.data_source == \"web_and_memory\""
        content: |
          Combined mode:
          1. Check memory-server for cached data first
          2. If cache miss, search web via duckduckgo
          3. Store findings in memory-server
          4. Return comprehensive results
      
      - if: "settings.data_source == \"offline\""
        content: |
          Offline mode:
          - Use training data only
          - No external tools
          - Note limitations to user
```

## Best Practices

### Write Clear Conditions

**Good:**
```yaml
instructions:
  - if: "settings.mode == \"expert\""
    content: |
      Expert mode instructions
```

**Bad:**
```yaml
instructions:
  - if: "settings.m == \"e\""
    content: |
      Unclear what this checks
```

### Always Include Default

**Good:**
```yaml
instructions:
  - if: "settings.expert_mode == true"
    content: |
      Expert instructions
  
  - |
      DEFAULT - Standard instructions
```

**Bad:**
```yaml
instructions:
  - if: "settings.expert_mode == true"
    content: |
      Expert instructions
  
  # No default! What happens in standard mode?
```

### Keep Conditions Simple

**Good:**
```yaml
instructions:
  - if: "settings.expert_mode == true"
    content: |
      Clear, simple condition
  
  - if: "settings.format == \"json\""
    content: |
      Clear equality check
```

### Document Complex Settings

```yaml
settings:
  - name: priority
    type: string
    title: Priority Level
    description: Request priority (low, normal, high, urgent). Affects response time and escalation rules.
    defaultValue: normal

agents:
  support:
    instructions:
      # High priority = Immediate detailed response
      - if: "settings.priority == \"high\""
        content: |
          High priority request - provide immediate response
      
      # Default = Standard response
      - |
          Standard mode - balanced response
```

## Testing Conditional Instructions

### Test Each Path

For this configuration:
```yaml
settings:
  - name: mode
    type: string
    title: Mode
    description: Operation mode
    defaultValue: standard

agents:
  assistant:
    instructions:
      - if: "settings.mode == \"advanced\""
        content: |
          Advanced instructions
      - if: "settings.mode == \"simple\""
        content: |
          Simple instructions
      - |
          Standard instructions
```

**Test cases:**
1. Set `mode = "advanced"` → Should see advanced instructions + standard
2. Set `mode = "simple"` → Should see simple instructions + standard
3. Set `mode = "standard"` → Should see standard instructions only

## Common Pitfalls

### Missing Default

❌ **Problem:**
```yaml
instructions:
  - if: "settings.expert_mode == true"
    content: |
      Expert instructions
  # No default!
```

✅ **Solution:**
```yaml
instructions:
  - if: "settings.expert_mode == true"
    content: |
      Expert instructions
  - |
      Standard instructions (default)
```

### Wrong Comparison Operator

❌ **Problem:**
```yaml
instructions:
  - if: "settings.format = \"json\""
    content: |
      Single = is assignment, not comparison!
```

✅ **Solution:**
```yaml
instructions:
  - if: "settings.format == \"json\""
    content: |
      Double == for equality check
```

### Attempting Unsupported Operators

❌ **Problem:**
```yaml
instructions:
  - if: "!settings.verbose"
    content: |
      Negation not supported!
  - if: "settings.a && settings.b"
    content: |
      Logical operators not supported!
```

✅ **Solution:**
```yaml
instructions:
  - if: "settings.verbose == true"
    content: |
      Verbose instructions
  - |
      Non-verbose behavior in default
```

## Next Steps

- **[Settings](settings.md)** - Complete settings documentation
- **[Configuration Reference](../agents/configuration.md)** - Full schema
- **[Agent Examples](../agents/examples.md)** - Teams using conditionals
- **[Multi-Agent Workflows](multi-agent.md)** - Advanced patterns

---

**Related:**  
[Settings](settings.md) | [Configuration](../agents/configuration.md) | [Creating Teams](../agents/creating-agents.md)
