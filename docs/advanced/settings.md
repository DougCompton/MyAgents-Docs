# Settings

Runtime configuration that allows users to customize team and agent behavior without code changes.

## What are Settings?

**Settings** are configurable options that:

- 🎛️ Give users control over team behavior
- 🔄 Change at runtime (no code deployment needed)
- 📝 Integrate with conditional instructions
- 🎯 Enable feature flags and modes
- ⚙️ Customize output formats and preferences

## Basic Settings

### Simple Boolean Toggle

```yaml
settings:
  - name: verbose_mode
    type: boolean
    title: Verbose Output
    description: Include detailed explanations in responses
    defaultValue: false
```

**Usage in agent:**
```yaml
agents:
  assistant:
    instructions:
      - "settings.verbose_mode": |
          VERBOSE MODE
          - Provide detailed explanations
          - Show reasoning process
          - Include examples and context
      
      - |
          STANDARD MODE
          - Provide concise responses
          - Focus on key information
```

### String Option

```yaml
settings:
  - name: output_format
    type: string
    title: Output Format
    description: Format for generated content
    defaultValue: markdown
```

**Usage in agent:**
```yaml
agents:
  writer:
    instructions:
      - "settings.output_format == 'markdown'": |
          Format output in Markdown:
          - Use ## for headings
          - Use **bold** and *italic*
          - Use bullet points and numbered lists
          - Use code blocks with ```
      
      - "settings.output_format == 'html'": |
          Format output in HTML:
          - Use <h2> for headings
          - Use <strong> and <em>
          - Use <ul> and <ol> for lists
          - Use <pre><code> for code
      
      - "settings.output_format == 'plain_text'": |
          Format output in plain text:
          - Use simple headings with caps
          - No formatting markup
          - Use dashes for lists
          - Indent code examples
```

### Numeric Setting

```yaml
settings:
  - name: max_results
    type: number
    title: Maximum Results
    description: Number of results to return
    defaultValue: 10
```

**Usage in agent:**
```yaml
agents:
  searcher:
    toolsets:
      - product-catalog
    instructions:
      - |
          Search products and return up to {settings.max_results} results.
          Present top matches based on relevance and quality.
```

## Setting Types

### Boolean Settings

**Use for:**
- Feature toggles
- Mode switches
- Enable/disable options

**Example - Feature Flags:**
```yaml
settings:
  - name: seo_mode
    type: boolean
    title: SEO Optimization
    description: Optimize content for search engines
    defaultValue: false
  
  - name: include_images
    type: boolean
    title: Include Images
    description: Add image suggestions to content
    defaultValue: true
  
  - name: fact_check
    type: boolean
    title: Fact Checking
    description: Verify claims against multiple sources
    defaultValue: false
```

**Agent usage:**
```yaml
content_writer:
  instructions:
    - "settings.seo_mode": |
        Apply SEO optimization:
        - Target keyword in title and headings
        - Meta description under 160 chars
        - Internal linking opportunities
        - Image alt text
    
    - "settings.include_images": |
        Suggest relevant images:
        - Describe ideal image for each section
        - Provide alt text
        - Note image placement
    
    - "settings.fact_check": |
        Fact-check all claims:
        - Search for authoritative sources
        - Cross-reference information
        - Include citations
    
    - |
        Create engaging content following standard guidelines.
```

### String Settings

**Use for:**
- Predefined options
- Output formats
- Style preferences
- Modes with multiple options

**Example - Style Preferences:**
```yaml
settings:
  - name: tone
    type: string
    title: Content Tone
    description: Writing style and formality
    defaultValue: conversational
  
  - name: length
    type: string
    title: Response Length
    description: Desired response length
    defaultValue: moderate
```

**Agent usage:**
```yaml
assistant:
  instructions:
    # Tone variations
    - "settings.tone == 'professional'": |
        Professional tone:
        - Formal language
        - Third-person perspective
        - Industry terminology
        - Authoritative voice
    
    - "settings.tone == 'conversational'": |
        Conversational tone:
        - Friendly, approachable
        - Second-person ("you")
        - Simple language
        - Relatable examples
    
    - "settings.tone == 'friendly'": |
        Friendly tone:
        - Warm and welcoming
        - Personal touches
        - Encouraging language
        - Empathetic responses
    
    - "settings.tone == 'academic'": |
        Academic tone:
        - Scholarly language
        - Evidence-based
        - Formal citations
        - Objective analysis
    
    - "settings.tone == 'humorous'": |
        Humorous tone:
        - Light and entertaining
        - Appropriate jokes
        - Witty observations
        - Playful language
    
    # Length variations
    - "settings.length == 'brief'": |
        Keep responses brief (1-2 paragraphs).
        Focus on essential information only.
    
    - "settings.length == 'moderate'": |
        Provide moderate detail (3-5 paragraphs).
        Include key points with some elaboration.
    
    - "settings.length == 'detailed'": |
        Provide comprehensive detail (6+ paragraphs).
        Include examples, context, and thorough explanations.
```

### Number Settings

**Use for:**
- Limits and thresholds
- Quantity preferences
- Numeric parameters

**Example - Numeric Controls:**
```yaml
settings:
  - name: max_results
    type: number
    title: Maximum Results
    description: Maximum number of search results
    defaultValue: 10
  
  - name: price_limit
    type: number
    title: Price Limit
    description: Maximum price in USD
    defaultValue: 100
  
  - name: target_word_count
    type: number
    title: Target Word Count
    description: Desired article length
    defaultValue: 1500
```

**Agent usage:**
```yaml
shopping_agent:
  instructions:
    - |
        Product recommendations:
        - Return up to {settings.max_results} products
        - Filter out items over ${settings.price_limit}
        - Sort by relevance and value
        - Present in order of match quality
```

## Conditional Instructions with Settings

### Simple Conditions

**Check if setting is true:**
```yaml
instructions:
  - "settings.expert_mode": |
      Expert mode active - provide technical details
```

**Check if setting is false:**
```yaml
instructions:
  - "!settings.expert_mode": |
      Standard mode - use accessible language
```

**Check string equality:**
```yaml
instructions:
  - "settings.format == 'json'": |
      Output in JSON format
```

**Check string inequality:**
```yaml
instructions:
  - "settings.format != 'json'": |
      Output in non-JSON format
```

### Complex Conditions

**Multiple conditions (AND):**
```yaml
instructions:
  - "settings.expert_mode && settings.verbose": |
      Expert + Verbose: Provide comprehensive technical detail
```

**Multiple conditions (OR):**
```yaml
instructions:
  - "settings.format == 'markdown' || settings.format == 'html'": |
      Use markup formatting (either Markdown or HTML)
```

**Nested conditions:**
```yaml
instructions:
  - "settings.expert_mode && (settings.format == 'json' || settings.format == 'yaml')": |
      Expert mode with structured data output
```

**Negation with complex logic:**
```yaml
instructions:
  - "!settings.simple_mode && settings.detail_level == 'high'": |
      Advanced mode with high detail
```

### Setting Comparisons

**Numeric comparisons:**
```yaml
instructions:
  - "settings.max_results > 20": |
      Large result set requested - include pagination info
  
  - "settings.price_limit < 50": |
      Budget-conscious search - prioritize value options
```

**String matching:**
```yaml
instructions:
  - "settings.tone == 'professional' || settings.tone == 'academic'": |
      Formal writing style required
```

## Common Setting Patterns

### Feature Flags

Enable/disable functionality:

```yaml
settings:
  - name: enable_web_search
    type: boolean
    title: Enable Web Search
    description: Allow searching the web for current information
    defaultValue: true
  
  - name: enable_notifications
    type: boolean
    title: Enable Notifications
    description: Send push notifications for updates
    defaultValue: false
  
  - name: enable_memory
    type: boolean
    title: Enable Memory
    description: Remember preferences and context
    defaultValue: true
```

**Agent usage:**
```yaml
assistant:
  toolsets:
    - duckduckgo
    - notification-server
    - memory-server
  instructions:
    - "settings.enable_web_search": |
        Web search enabled - use duckduckgo for current information
    
    - "!settings.enable_web_search": |
        Web search disabled - use training data only
    
    - "settings.enable_notifications": |
        Send important updates via notification-server
    
    - "settings.enable_memory": |
        Store user preferences and context in memory-server
    
    - |
        Core assistant functionality
```

### User Preferences

Store user-specific preferences:

```yaml
settings:
  - name: temperature_unit
    type: string
    title: Temperature Unit
    description: Display temperatures in Fahrenheit or Celsius
    defaultValue: F
  
  - name: date_format
    type: string
    title: Date Format
    description: Preferred date display format
    defaultValue: MM/DD/YYYY
  
  - name: timezone
    type: string
    title: Timezone
    description: User timezone for scheduling
    defaultValue: America/New_York
```

### Dietary Restrictions

Common in e-commerce and food teams:

```yaml
settings:
  - name: dietary_restrictions
    type: string
    title: Dietary Restrictions
    description: Filter products and recipes by dietary needs
    defaultValue: none
  
  - name: allergies
    type: string
    title: Allergies
    description: Food allergies to avoid
    defaultValue: none
```

**Agent usage:**
```yaml
meal_planner:
  toolsets:
    - product-catalog
  instructions:
    - "settings.dietary_restrictions != 'none'": |
        Filter all recipes and products by: {settings.dietary_restrictions}
        Clearly label dietary attributes.
        Verify ingredients meet restrictions.
    
    - "settings.allergies != 'none'": |
        CRITICAL: Exclude all products containing: {settings.allergies}
        Double-check ingredient lists.
        Warn if allergen information unavailable.
    
    - |
        Provide meal plans and shopping lists.
```

### Experience Level

Adjust complexity based on user expertise:

```yaml
settings:
  - key: experience_level
    type: string
    label: Experience Level
    description: User's expertise in this domain
    allowed_values:
      - beginner
      - intermediate
      - advanced
      - expert
    default_value: intermediate
```

**Agent usage:**
```yaml
instructor:
  instructions:
    - "settings.experience_level == 'beginner'": |
        Beginner level:
        - Define all technical terms
        - Start with fundamentals
        - Provide step-by-step guidance
        - Include lots of examples
        - Avoid jargon
    
    - "settings.experience_level == 'intermediate'": |
        Intermediate level:
        - Assume basic knowledge
        - Focus on practical applications
        - Introduce advanced concepts gradually
        - Provide context and rationale
    
    - "settings.experience_level == 'advanced'": |
        Advanced level:
        - Assume strong foundation
        - Focus on nuance and edge cases
        - Discuss trade-offs and alternatives
        - Reference best practices
    
    - "settings.experience_level == 'expert'": |
        Expert level:
        - Discuss advanced techniques
        - Debate architectural decisions
        - Explore cutting-edge approaches
        - Challenge assumptions
```

## Best Practices

### Setting Design

**Clear Labels:**
```yaml
# Good
label: Temperature Unit
label: Output Format
label: Enable Web Search

# Bad
label: Temp
label: Format
label: Search
```

**Descriptive Descriptions:**
```yaml
# Good
description: Display temperatures in Fahrenheit or Celsius

# Bad
description: Temperature setting
```

**Sensible Defaults:**
```yaml
# Good - Safe, common defaults
default_value: false  # for feature flags
default_value: 10     # for reasonable limits
default_value: markdown  # for common format

# Bad - Unusual defaults
default_value: true   # for experimental features
default_value: 1000   # for unreasonable limits
```

**Limited Options:**
```yaml
# Good - Manageable set of options
allowed_values:
  - small
  - medium
  - large

# Bad - Too many options
allowed_values:
  - option1
  - option2
  ... (20 more options)
```

### Instruction Organization

**Group related conditions:**
```yaml
instructions:
  # Tone settings
  - "settings.tone == 'professional'": |
      Professional tone instructions
  - "settings.tone == 'casual'": |
      Casual tone instructions
  
  # Format settings
  - "settings.format == 'json'": |
      JSON format instructions
  - "settings.format == 'yaml'": |
      YAML format instructions
  
  # Default
  - |
      Standard instructions
```

**Default fallback:**
Always include a default instruction (empty condition):

```yaml
instructions:
  - "settings.expert_mode": |
      Expert instructions
  
  - |
      DEFAULT - Standard instructions (always present)
```

### Setting Validation

**Use allowed_values for strings:**
```yaml
# Good - Constrained options
settings:
  - key: color
    type: string
    allowed_values:
      - red
      - blue
      - green
    default_value: blue

# Bad - No constraints
settings:
  - key: color
    type: string
    # User can enter anything
    default_value: blue
```

**Set reasonable ranges for numbers:**
```yaml
# Add validation in agent instructions
instructions:
  - |
      Validate settings:
      - max_results must be between 1 and 100
      - price_limit must be positive
      - If invalid, use default and notify user
```

## Advanced Patterns

### Dependent Settings

Settings that affect each other:

```yaml
settings:
  - key: research_mode
    type: boolean
    label: Research Mode
    description: Conduct thorough research before responding
    default_value: false
  
  - key: max_sources
    type: number
    label: Maximum Sources
    description: Number of sources to check (research mode only)
    default_value: 5
```

**Agent usage:**
```yaml
researcher:
  toolsets:
    - duckduckgo
  instructions:
    - "settings.research_mode": |
        RESEARCH MODE ACTIVE
        - Search up to {settings.max_sources} sources
        - Cross-reference information
        - Provide citations
        - Note conflicting information
    
    - |
        Standard response (max_sources setting ignored)
```

### Dynamic Settings

Settings that change based on context:

```yaml
settings:
  - key: auto_mode
    type: boolean
    label: Auto Mode
    description: Automatically adjust settings based on request
    default_value: false
  
  - key: detail_level
    type: string
    label: Detail Level
    description: Response detail level (ignored in auto mode)
    allowed_values:
      - brief
      - normal
      - detailed
    default_value: normal
```

**Agent usage:**
```yaml
adaptive_assistant:
  instructions:
    - "settings.auto_mode": |
        AUTO MODE
        - Detect request complexity
        - Adjust detail level automatically:
          - Simple question → brief response
          - Complex question → detailed response
          - Research request → exhaustive response
    
    - "!settings.auto_mode && settings.detail_level == 'brief'": |
        Brief responses (user preference)
    
    - "!settings.auto_mode && settings.detail_level == 'detailed'": |
        Detailed responses (user preference)
    
    - |
        Normal detail level
```

### Setting Profiles

Predefined combinations of settings:

```yaml
settings:
  - key: profile
    type: string
    label: User Profile
    description: Predefined setting combinations
    allowed_values:
      - student
      - professional
      - researcher
      - casual
    default_value: casual
```

**Agent usage:**
```yaml
assistant:
  instructions:
    - "settings.profile == 'student'": |
        STUDENT PROFILE
        - Educational tone
        - Detailed explanations
        - Include examples
        - Cite sources
        - Avoid jargon
    
    - "settings.profile == 'professional'": |
        PROFESSIONAL PROFILE
        - Business-focused
        - Concise and efficient
        - Industry terminology okay
        - Actionable insights
        - Data-driven
    
    - "settings.profile == 'researcher'": |
        RESEARCHER PROFILE
        - Academic rigor
        - Comprehensive citations
        - Multiple perspectives
        - Thorough analysis
        - Primary sources preferred
    
    - "settings.profile == 'casual'": |
        CASUAL PROFILE
        - Friendly tone
        - Conversational style
        - Simple explanations
        - Practical focus
```

## Next Steps

- **[Conditional Instructions](conditional-instructions.md)** - Advanced conditional logic
- **[Configuration Reference](../agents/configuration.md)** - Complete settings schema
- **[Agent Examples](../agents/examples.md)** - Teams using settings
- **[Multi-Agent Workflows](multi-agent.md)** - Complex team patterns

---

**Related:**  
[Conditional Instructions](conditional-instructions.md) | [Configuration](../agents/configuration.md) | [Creating Teams](../agents/creating-agents.md)
