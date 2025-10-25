# Getting Started

This guide will help you create your first AI agent in just a few minutes.

## Prerequisites

Before you begin, ensure you have:

- Access to the MyAgents system
- A text editor for creating JSON files
- Basic understanding of JSON format

## Your First Agent: Hello World

Let's create a simple conversational agent that introduces itself.

### Step 1: Create the Agent File

Create a new file named `hello-agent.json` in the `apps/api/src/agents/` directory:

```json
{
  "id": "hello-agent",
  "name": "Hello Agent",
  "description": "A friendly agent that introduces itself and answers basic questions",
  "interactive": true,
  "agents": [
    {
      "name": "greeter",
      "description": "Greets users and provides information",
      "instructions": [
        {
          "content": "You are a friendly AI assistant named Hello Agent. Your purpose is to greet users warmly and help them understand what you can do. Keep your responses concise and friendly."
        }
      ]
    }
  ]
}
```

### Step 2: Understand the Structure

Let's break down what each property does:

- **`id`**: Unique identifier for your agent (use lowercase with hyphens)
- **`name`**: Display name shown to users
- **`description`**: Brief explanation of what the agent does
- **`interactive`**: Set to `true` so the agent maintains context across messages
- **`agents`**: Array of personas (specialized roles within your agent)

### Step 3: Test Your Agent

1. Save the file in `apps/api/src/agents/hello-agent.json`
2. Restart the API service (if required for agent reload)
3. Use the chat interface to interact with your agent
4. Try saying: "Hello! What can you help me with?"

## Adding Tools to Your Agent

Now let's enhance your agent with web search capabilities using the DuckDuckGo tool.

### Updated Agent with Tools

```json
{
  "id": "hello-agent",
  "name": "Hello Agent",
  "description": "A friendly agent that can search the web and answer questions",
  "interactive": true,
  "agents": [
    {
      "name": "greeter",
      "description": "Greets users and provides information using web search",
      "instructions": [
        {
          "content": "You are a friendly AI assistant. Greet users warmly and help them find information. When asked about current events or factual information, use the web search tool to find accurate, up-to-date answers."
        }
      ],
      "toolsets": ["duckduckgo"]
    }
  ]
}
```

### What Changed?

We added the `"toolsets": ["duckduckgo"]` property, which gives your agent access to web search capabilities.

### Test the Enhanced Agent

Try asking:
- "What's the weather like today?"
- "Tell me about recent AI developments"
- "Search for the latest news about electric cars"

## Adding Configuration Settings

Let's make your agent more flexible with settings.

### Agent with Settings

```json
{
  "id": "hello-agent",
  "name": "Hello Agent",
  "description": "A configurable agent that adapts its communication style",
  "interactive": true,
  "settings": [
    {
      "name": "formalMode",
      "title": "Formal Communication",
      "type": "bool",
      "description": "Use formal, professional language",
      "defaultValue": false
    },
    {
      "name": "responseLength",
      "title": "Response Length",
      "type": "string",
      "description": "Preferred response length (brief, moderate, detailed)",
      "defaultValue": "moderate"
    }
  ],
  "agents": [
    {
      "name": "greeter",
      "description": "Greets users with configurable communication style",
      "instructions": [
        {
          "content": "You are a friendly AI assistant. Keep responses conversational and approachable."
        },
        {
          "content": "\nCOMMUNICATION STYLE: Use formal, professional language. Address users respectfully and maintain business-appropriate tone.",
          "if": "setting.formalMode == true"
        },
        {
          "content": "\nRESPONSE LENGTH: Keep responses very brief, typically 1-2 sentences.",
          "if": "setting.responseLength == \"brief\""
        },
        {
          "content": "\nRESPONSE LENGTH: Provide detailed, comprehensive responses with examples and explanations.",
          "if": "setting.responseLength == \"detailed\""
        }
      ],
      "toolsets": ["duckduckgo"]
    }
  ]
}
```

### Understanding Settings

**Settings** allow you to configure agent behavior without changing code:

- **Boolean settings** (`type: "bool"`) - Toggle features on/off
- **String settings** (`type: "string"`) - Choose between predefined options
- **Number settings** (`type: "number"`) - Set numeric thresholds or limits

**Conditional instructions** use the `"if"` property to apply instructions only when certain settings are active.

## Multi-Persona Agent Example

For more complex workflows, create agents with multiple specialized personas.

### Blog Writing Team

```json
{
  "id": "blog-team",
  "name": "Blog Writing Team",
  "description": "Coordinates research, writing, and editing for blog content",
  "interactive": true,
  "agents": [
    {
      "name": "coordinator",
      "description": "Manages the blog writing workflow",
      "instructions": [
        {
          "content": "You are a project coordinator for a blog writing team. When users request a blog post:\n1. Use transfer_task to delegate to 'researcher' for gathering information\n2. Then delegate to 'writer' for creating the draft\n3. Finally delegate to 'editor' for review and improvements\n\nCoordinate the process and present the final result to the user."
        }
      ],
      "subAgents": ["researcher", "writer", "editor"]
    },
    {
      "name": "researcher",
      "description": "Gathers information from the web",
      "instructions": [
        {
          "content": "You are a researcher. Search for current, accurate information on the requested topic. Organize your findings clearly and cite sources."
        }
      ],
      "toolsets": ["duckduckgo"]
    },
    {
      "name": "writer",
      "description": "Creates blog post content",
      "instructions": [
        {
          "content": "You are a skilled writer. Create an engaging blog post based on the research provided. Use clear structure with introduction, body sections with headers, and conclusion. Write in a professional but approachable tone."
        }
      ]
    },
    {
      "name": "editor",
      "description": "Reviews and improves content",
      "instructions": [
        {
          "content": "You are an editor. Review the blog post for grammar, clarity, flow, and engagement. Make improvements and explain your key changes."
        }
      ]
    }
  ]
}
```

### Key Concepts

- **Coordinator persona**: Orchestrates the workflow using `subAgents`
- **Specialist personas**: Each handles a specific task (research, write, edit)
- **Task delegation**: Uses the `transfer_task` tool to pass work between personas

## Next Steps

Congratulations! You've created your first agents. Here's what to explore next:

### Learn More About Agents
- [Agent Overview](agents/overview.md) - Deep dive into agent architecture
- [Creating Agents](agents/creating-agents.md) - Comprehensive agent creation guide
- [Configuration Reference](agents/configuration.md) - All configuration options

### Explore Tools
- [Available Tools](tools/available-tools.md) - Browse all available MCP tools
- [Using Tools](tools/using-tools.md) - Best practices for tool integration
- [Creating Tools](tools/creating-tools.md) - Build your own custom tools

### Advanced Features
- [Settings & Configuration](advanced/settings.md) - Master runtime configuration
- [Conditional Instructions](advanced/conditional-instructions.md) - Dynamic agent behavior
- [Multi-Persona Workflows](advanced/multi-persona.md) - Complex orchestration patterns

### Examples
- [Agent Examples](agents/examples.md) - Real-world templates for common use cases

## Quick Reference

### Minimal Agent Structure
```json
{
  "id": "agent-id",
  "name": "Agent Name",
  "description": "What the agent does",
  "agents": [
    {
      "name": "persona_name",
      "description": "What this persona does",
      "instructions": [
        {"content": "Detailed instructions for the agent..."}
      ]
    }
  ]
}
```

### Agent with Tools
```json
{
  "id": "agent-id",
  "name": "Agent Name",
  "description": "What the agent does",
  "agents": [
    {
      "name": "persona_name",
      "description": "What this persona does",
      "instructions": [{"content": "Instructions..."}],
      "toolsets": ["duckduckgo", "memory-server"]
    }
  ]
}
```

### Interactive vs Non-Interactive

**Interactive** (`"interactive": true`):
- Agent stays active across multiple messages
- Use for: conversations, iterative work, multi-step tasks
- Examples: Shopping assistants, content creation, consulting

**Non-Interactive** (omit property or set to `false`):
- Agent returns to switchboard after completing request
- Use for: quick queries, single transactions, calculations
- Examples: Weather lookups, currency conversion, simple data retrieval

## Common Pitfalls to Avoid

### ❌ Incorrect JSON Syntax
```json
{
  "interactive": "true"  // Wrong - string instead of boolean
}
```

### ✅ Correct JSON Syntax
```json
{
  "interactive": true  // Correct - boolean value
}
```

### ❌ Missing Required Fields
```json
{
  "id": "my-agent",
  "agents": []  // Missing "name" and "description"
}
```

### ✅ All Required Fields
```json
{
  "id": "my-agent",
  "name": "My Agent",
  "description": "What it does",
  "agents": [...]
}
```

## Troubleshooting

### Agent Doesn't Load
- Check JSON syntax (use a JSON validator)
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
- Use escaped quotes for string values: `"setting.mode == \"value\""`

## Getting Help

If you encounter issues:

1. **Check the documentation** - Review relevant sections in this guide
2. **Validate your JSON** - Use an online JSON validator
3. **Check logs** - Look for error messages in server logs
4. **Review examples** - Compare your agent to working examples
5. **Contact support** - Reach out to your system administrator

---

**Next:** [Understanding Agents](agents/overview.md) | [Available Tools](tools/available-tools.md)

