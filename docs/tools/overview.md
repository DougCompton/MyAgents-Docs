# Understanding Tools

Tools extend your agent's capabilities by connecting to external systems and services through the **Model Context Protocol (MCP)**.

## What are MCP Tools?

**MCP (Model Context Protocol)** is a standardized way for AI agents to interact with external systems, APIs, and data sources.

### Why Tools Matter

Without tools, agents can only:
- Process and analyze text
- Generate responses based on training data
- Follow instructions

With tools, agents can:
- 🔍 **Search the web** for current information
- 💾 **Store and retrieve data** persistently
- 📅 **Manage calendars** and schedules
- 🔔 **Send notifications** to users
- 🌤️ **Get weather data** and forecasts
- 📊 **Access external APIs** for specialized data

## How Tools Work

### Tool Architecture

```
┌─────────────────────────────────────────┐
│           AI Agent                      │
│  ┌─────────────────────────────────┐   │
│  │  Persona with toolsets          │   │
│  │  "toolsets": ["duckduckgo"]     │   │
│  └─────────────┬───────────────────┘   │
└────────────────┼───────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│       MCP Registry                      │
│  (Maps toolset names to servers)        │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│       MCP Server                        │
│  (Executes tool operations)             │
│  - duckduckgo search                    │
│  - Returns results                      │
└─────────────────────────────────────────┘
```

### Tool Execution Flow

1. **Agent decides** to use a tool based on instructions
2. **Tool call** is made with specific parameters
3. **MCP Server** executes the operation
4. **Results** are returned to the agent
5. **Agent processes** results and continues

### Example: Web Search

User: "What's the latest news about electric cars?"

```
1. Agent receives question
2. Agent decides to use web search tool
3. Calls: search("latest electric car news")
4. DuckDuckGo searches and returns results
5. Agent synthesizes results into response
6. User receives answer with sources
```

## Tool Types

### Built-in Tools

System tools automatically available to agents:

#### Delegation Tools
- **`switch_agent`** - Switch to a different agent
- **`transfer_task`** - Delegate task to sub-persona

These are automatically available and don't need to be specified in `toolsets`.

### External Tools

External services accessed via MCP servers:

#### Information & Search
- **`duckduckgo`** - Web search
- **`weather-tool`** - Weather information

#### Data & Storage
- **`memory-server`** - Persistent data storage
- **`notification-server`** - Push notifications

#### Integrations
- **`google-calendar`** - Google Calendar access (requires OAuth)

See [Available Tools](available-tools.md) for complete details.

## Adding Tools to Agents

### Simple Tool Addition

Add tools to the persona that needs them:

```json
{
  "name": "researcher",
  "description": "Researches topics using web search",
  "instructions": [
    {
      "content": "You research topics thoroughly using web search. Always cite sources and verify information from multiple sources."
    }
  ],
  "toolsets": ["duckduckgo"]
}
```

### Multiple Tools

Personas can use multiple tools:

```json
{
  "name": "shopping_assistant",
  "description": "Helps with shopping and remembers preferences",
  "instructions": [
    {
      "content": "You help users shop online. Search for products and remember user preferences for future recommendations."
    }
  ],
  "toolsets": ["duckduckgo", "memory-server"]
}
```

### Tool Exclusions (Advanced)

Conditionally exclude specific tools based on settings:

```json
{
  "name": "file_manager",
  "description": "Manages files with permission controls",
  "toolsets": [
    {
      "name": "filesystem",
      "exclude": [
        {
          "tool": "delete",
          "if": "setting.readOnlyMode == true"
        }
      ]
    }
  ]
}
```

This prevents the `delete` tool from being available when in read-only mode.

## Best Practices

### Tool Selection

**Only add tools that personas actually need:**

✅ Good - Specific tool needs:
```json
{
  "name": "researcher",
  "toolsets": ["duckduckgo"]  // Needs web search
}
```

❌ Bad - Unnecessary tools:
```json
{
  "name": "coordinator",
  "toolsets": ["duckduckgo", "memory-server", "weather-tool"]
  // Coordinator just delegates, doesn't need tools
}
```

### Tool Instructions

**Tell agents when and how to use tools:**

```json
{
  "instructions": [
    {
      "content": "When users ask about current events or recent information, use the web search tool to find accurate, up-to-date answers. Always cite your sources with URLs."
    }
  ],
  "toolsets": ["duckduckgo"]
}
```

### Tool Reliability

**Handle tool failures gracefully:**

```json
{
  "content": "When using tools:\n- Always verify tool results before presenting to users\n- If a tool fails, explain the issue and try alternative approaches\n- For web search, cross-reference multiple sources\n- Note when information might be outdated or unreliable"
}
```

### Tool Performance

**Consider tool costs and limits:**

- Some tools have API rate limits
- External API calls add latency
- Tools may have usage costs
- Design agents to minimize unnecessary tool calls

## Tool Use Cases

### Research & Information Gathering

```json
{
  "name": "market_researcher",
  "description": "Researches market trends and competitors",
  "instructions": [
    {
      "content": "Research market trends, competitor analysis, and industry insights. Use web search to find recent data, reports, and news. Synthesize findings into clear, actionable insights."
    }
  ],
  "toolsets": ["duckduckgo"]
}
```

### Data Persistence

```json
{
  "name": "preference_manager",
  "description": "Manages user preferences and history",
  "instructions": [
    {
      "content": "Track user preferences, shopping history, and personalization data. Use memory-server to store and retrieve this information across sessions. Respect user privacy and data retention policies."
    }
  ],
  "toolsets": ["memory-server"]
}
```

### Calendar & Scheduling

```json
{
  "name": "meeting_scheduler",
  "description": "Schedules meetings and manages calendar",
  "instructions": [
    {
      "content": "Help users schedule meetings, check availability, and manage their calendar. Use google-calendar to access and modify calendar events. Always confirm changes before making them."
    }
  ],
  "toolsets": ["google-calendar"]
}
```

### Notifications & Alerts

```json
{
  "name": "reminder_agent",
  "description": "Sends reminders and notifications",
  "instructions": [
    {
      "content": "Create and manage reminders for users. Use notification-server to send push notifications at appropriate times. Respect user notification preferences and quiet hours."
    }
  ],
  "toolsets": ["notification-server", "memory-server"]
}
```

## Tool Security

### Permission Management

Some tools require authentication or permissions:

- **OAuth Tools**: Users must authenticate (e.g., `google-calendar`)
- **Permission Scopes**: Tools request specific permissions
- **User Consent**: Users approve tool access before use

### Data Privacy

When using tools that access user data:

1. **Respect privacy** - Only access data necessary for the task
2. **Secure storage** - Use proper encryption and access controls
3. **Data retention** - Follow retention policies
4. **User control** - Allow users to view and delete their data

### Error Handling

Handle tool errors appropriately:

```json
{
  "content": "Error handling for tools:\n- Network failures: Inform user and suggest retry\n- Permission errors: Guide user through authentication\n- Rate limits: Explain delay and provide alternatives\n- Invalid results: Validate data before using"
}
```

## Tool Development

Want to create custom tools? See:

- **[Creating Tools](creating-tools.md)** - Build custom MCP servers
- **[Tool Integration](using-tools.md)** - Advanced integration patterns

## Debugging Tools

### Tool Call Logging

When debugging tool issues:

1. Check server logs for tool call details
2. Verify tool parameters are correct
3. Test tool independently of agent
4. Review tool response format

### Common Issues

**Tool not found:**
- Verify toolset name spelling (case-sensitive)
- Check tool is registered in MCP registry
- Ensure server is running (for external tools)

**Tool fails:**
- Check network connectivity (for external tools)
- Verify API credentials and permissions
- Review rate limits and quotas
- Check tool-specific documentation

**Unexpected results:**
- Validate input parameters
- Review tool response format
- Check for API version mismatches
- Verify data formats and encoding

## Next Steps

- **[Available Tools](available-tools.md)** - Browse all available tools
- **[Using Tools](using-tools.md)** - Best practices and patterns
- **[Creating Tools](creating-tools.md)** - Build custom tools
- **[Tool Examples](../agents/examples.md)** - See tools in action

---

**Related:**  
[Agent Overview](../agents/overview.md) | [Available Tools](available-tools.md) | [Creating Agents](../agents/creating-agents.md)

