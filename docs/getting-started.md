# Getting Started

This guide will help you create your first AI agent team in just a few minutes.

## Prerequisites

Before you begin, ensure you have:

- Access to the MyAgents system
- A text editor for creating YAML files
- Basic understanding of YAML format

## Your First Team: Hello World

Let's create a simple team with one agent that introduces itself.

### Step 1: Create the Team File

Create a new file named `hello-team.yaml` in the `apps/api/src/agents/` directory:

```yaml
version: "1"
id: hello-team
name: Hello Team
description: A friendly team that introduces itself and answers basic questions
interactive: true
default_agent: greeter

agents:
  greeter:
    type: llm
    name: Greeter
    description: Greets users and provides information
    instructions:
      - |
        You are a friendly AI assistant named Hello Agent. 
        Your purpose is to greet users warmly and help them understand what you can do. 
        Keep your responses concise and friendly.
```

### Step 2: Understand the Structure

Let's break down what each property does:

- **`version`**: Schema version (always "1")
- **`id`**: Unique identifier for your team (use lowercase with hyphens)
- **`name`**: Display name shown to users
- **`description`**: Brief explanation of what the team does
- **`interactive`**: Set to `true` so the team maintains context across messages
- **`default_agent`**: Which agent handles requests by default
- **`agents`**: Dictionary of agents in the team

### Step 3: Test Your Team

1. Save the file in `apps/api/src/agents/hello-team.yaml`
2. Restart the API service (if required for team reload)
3. Use the chat interface to interact with your team
4. Try saying: "Hello! What can you help me with?"

## Adding Tools to Your Agent

Now let's enhance your agent with web search capabilities using the DuckDuckGo tool.

### Updated Team with Tools

```yaml
version: "1"
id: hello-team
name: Hello Team
description: A friendly team that can search the web and answer questions
interactive: true
default_agent: greeter

agents:
  greeter:
    type: llm
    name: Greeter
    description: Greets users and provides information using web search
    instructions:
      - |
        You are a friendly AI assistant. Greet users warmly and help them find information. 
        When asked about current events or factual information, use the web search tool 
        to find accurate, up-to-date answers.
    toolsets:
      - duckduckgo
```

### What Changed?

We added `toolsets: [duckduckgo]`, which gives your agent access to web search capabilities.

### Test the Enhanced Team

Try asking:
- "What's the weather like today?"
- "Tell me about recent AI developments"
- "Search for the latest news about electric cars"

## Adding Configuration Settings

Let's make your team more flexible with settings.

### Team with Settings

```yaml
version: "1"
id: hello-team
name: Hello Team
description: A configurable team that adapts its communication style
interactive: true
default_agent: greeter

settings:
  - name: formalMode
    title: Formal Communication
    type: bool
    description: Use formal, professional language
    defaultValue: false

  - name: responseLength
    title: Response Length
    type: string
    description: Preferred response length (brief, moderate, detailed)
    defaultValue: moderate

agents:
  greeter:
    type: llm
    name: Greeter
    description: Greets users with configurable communication style
    instructions:
      - |
        You are a friendly AI assistant. Keep responses conversational and approachable.
      
      - if: "settings.formalMode == true"
        content: |
          COMMUNICATION STYLE: Use formal, professional language. 
          Address users respectfully and maintain business-appropriate tone.
      
      - if: "settings.responseLength == \"brief\""
        content: |
          RESPONSE LENGTH: Keep responses very brief, typically 1-2 sentences.
      
      - if: "settings.responseLength == \"detailed\""
        content: |
          RESPONSE LENGTH: Provide detailed, comprehensive responses with examples and explanations.
    
    toolsets:
      - duckduckgo
```

### Understanding Settings

**Settings** allow you to configure team behavior without changing code:

- **Boolean settings** (`type: bool`) - Toggle features on/off
- **String settings** (`type: string`) - Choose between predefined options
- **Number settings** (`type: number`) - Set numeric thresholds or limits

**Conditional instructions** use the `if` property to apply instructions only when certain settings are active.

## Multi-Agent Team Example

For more complex workflows, create teams with multiple specialized agents.

### Blog Writing Team

```yaml
version: "1"
id: blog-team
name: Blog Writing Team
description: Coordinates research, writing, and editing for blog content
interactive: true
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Coordinator
    description: Manages the blog writing workflow
    instructions:
      - |
        You are a project coordinator for a blog writing team. When users request a blog post:
        
        1. Use transfer_task to delegate to 'researcher' for gathering information
        2. Then delegate to 'writer' for creating the draft
        3. Finally delegate to 'editor' for review and improvements
        
        Coordinate the process and present the final result to the user.
    sub_agents:
      - researcher
      - writer
      - editor
  
  researcher:
    type: llm
    name: Researcher
    description: Gathers information from the web
    instructions:
      - |
        You are a researcher. Search for current, accurate information on the requested topic. 
        Organize your findings clearly and cite sources.
    toolsets:
      - duckduckgo
  
  writer:
    type: llm
    name: Writer
    description: Creates blog post content
    instructions:
      - |
        You are a skilled writer. Create an engaging blog post based on the research provided. 
        Use clear structure with introduction, body sections with headers, and conclusion. 
        Write in a professional but approachable tone.
  
  editor:
    type: llm
    name: Editor
    description: Reviews and improves content
    instructions:
      - |
        You are an editor. Review the blog post for grammar, clarity, flow, and engagement. 
        Make improvements and explain your key changes.
```

### Key Concepts

- **Coordinator agent**: Orchestrates the workflow using `sub_agents`
- **Specialist agents**: Each handles a specific task (research, write, edit)
- **Task delegation**: Uses the `transfer_task` tool to pass work between agents

## Next Steps

Congratulations! You've created your first agent teams. Here's what to explore next:

### Learn More About Teams
- [Agent Overview](agents/overview.md) - Deep dive into team architecture
- [Creating Teams](agents/creating-agents.md) - Comprehensive team creation guide
- [Configuration Reference](agents/configuration.md) - All configuration options

### Explore Tools
- [Available Tools](tools/available-tools.md) - Browse all available MCP tools
- [Using Tools](tools/using-tools.md) - Best practices for tool integration

### Advanced Features
- [Settings & Configuration](advanced/settings.md) - Master runtime configuration
- [Conditional Instructions](advanced/conditional-instructions.md) - Dynamic agent behavior
- [Multi-Agent Workflows](advanced/multi-agent.md) - Complex team orchestration patterns

### Examples
- [Team Examples](agents/examples.md) - Real-world templates for common use cases

## Quick Reference

### Minimal Team Structure
```yaml
version: "1"
id: team-id
name: Team Name
description: What the team does
default_agent: agent_name

agents:
  agent_name:
    type: llm
    name: Agent Name
    description: What this agent does
    instructions:
      - "": Detailed instructions for the agent...
```

### Team with Tools
```yaml
version: "1"
id: team-id
name: Team Name
description: What the team does
default_agent: agent_name

agents:
  agent_name:
    type: llm
    name: Agent Name
    description: What this agent does
    instructions:
      - "": Instructions...
    toolsets:
      - duckduckgo
      - memory-server
```

### Interactive vs Non-Interactive

**Interactive** (`interactive: true`):
- Team stays active across multiple messages
- Use for: conversations, iterative work, multi-step tasks
- Examples: Shopping assistants, content creation, consulting

**Non-Interactive** (omit property or set to `false`):
- Team returns to switchboard after completing request
- Use for: quick queries, single transactions, calculations
- Examples: Weather lookups, currency conversion, simple data retrieval

## Common Pitfalls to Avoid

### ❌ Incorrect YAML Syntax
```yaml
interactive: "true"  # Wrong - string instead of boolean
```

### ✅ Correct YAML Syntax
```yaml
interactive: true  # Correct - boolean value
```

### ❌ Missing Required Fields
```yaml
version: "1"
id: my-team
agents:
  helper: {}  # Missing required fields
```

### ✅ All Required Fields
```yaml
version: "1"
id: my-team
name: My Team
description: What it does
default_agent: helper

agents:
  helper:
    type: llm
    name: Helper
    description: Helps users
    instructions:
      - "": Instructions here
```

## Troubleshooting

### Team Doesn't Load
- Check YAML syntax (use a YAML validator)
- Verify all required fields are present
- Check server logs for specific errors
- Ensure file is in `apps/api/src/agents/` directory

### Tools Don't Work
- Verify toolset name spelling (case-sensitive)
- Check that toolset is registered in system
- Review tool-specific configuration requirements

### Conditional Instructions Not Working
- Verify setting name matches exactly
- Check expression syntax: `setting.<name> == <value>`
- Use quotes for string values: `setting.mode == "value"`

## Getting Help

If you encounter issues:

1. **Check the documentation** - Review relevant sections in this guide
2. **Validate your YAML** - Use an online YAML validator
3. **Check logs** - Look for error messages in server logs
4. **Review examples** - Compare your team to working examples
5. **Contact support** - Reach out to your system administrator

---

**Next:** [Understanding Teams](agents/overview.md) | [Available Tools](tools/available-tools.md)
