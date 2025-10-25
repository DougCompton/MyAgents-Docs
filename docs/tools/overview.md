# Tools Overview

Tools extend agent capabilities by connecting them to external systems, data sources, and services.

## What are Tools?

**Tools** are integrations that allow agents to:

- 🔍 **Search the web** for current information
- 💾 **Store and retrieve** persistent data
- 📧 **Send notifications** to users
- 🗓️ **Manage calendars** and scheduling
- 🌤️ **Access real-time data** like weather
- 🛠️ **Integrate with APIs** and external services

Tools are based on the **Model Context Protocol (MCP)**, an open standard for connecting AI systems to data sources.

## How Tools Work

### Tool Integration

Agents access tools through the `toolsets` configuration property:

```yaml
agents:
  researcher:
    type: llm
    name: Research Assistant
    description: Conducts web research
    toolsets:
      - duckduckgo      # Web search tool
      - memory-server   # Data persistence tool
    instructions:
      - |
          Use duckduckgo to research topics.
          Store important findings in memory-server for future reference.
```

### Tool Execution Flow

```
┌──────────┐
│   User   │
│  Request │
└────┬─────┘
     │
     ▼
┌────────────────┐
│     Agent      │
│  (Processes    │
│   request)     │
└────┬───────────┘
     │
     │ Agent decides to use tool
     ▼
┌────────────────┐
│  Tool Call     │
│  (e.g.,        │
│   web search)  │
└────┬───────────┘
     │
     │ Tool returns results
     ▼
┌────────────────┐
│     Agent      │
│  (Processes    │
│   results)     │
└────┬───────────┘
     │
     ▼
┌────────────────┐
│   Response     │
│   to User      │
└────────────────┘
```

### Automatic Tool Access

When you add a toolset to an agent:

1. ✅ Agent automatically receives tool descriptions
2. ✅ Agent can invoke tools during processing
3. ✅ Agent receives tool results in context
4. ✅ Agent decides when to use which tools

**You don't need to:**
- ❌ Write tool invocation code
- ❌ Handle tool responses manually
- ❌ Manage tool authentication (handled by system)

## Tool Categories

### Information Gathering

Tools that retrieve external information:

- **Web Search** (`duckduckgo`) - Search the web for current information
- **Weather** (`weather-tool`) - Get weather conditions and forecasts
- **Calendar** (`google-calendar`) - Access calendar events and availability

**Example:**
```yaml
researcher:
  toolsets:
    - duckduckgo
    - weather-tool
  instructions:
    - |
        Research topics using web search.
        Check weather when providing outdoor recommendations.
```

### Data Persistence

Tools that store and retrieve data:

- **Memory Server** (`memory-server`) - Key-value storage for user data and preferences
- **Database Connectors** - Integration with databases (custom implementations)

**Example:**
```yaml
shopping_assistant:
  toolsets:
    - memory-server
  instructions:
    - |
        Store user preferences (dietary restrictions, favorite brands).
        Retrieve preferences before making product recommendations.
```

### Communication

Tools that send messages and notifications:

- **Notification Server** (`notification-server`) - Push notifications to users
- **Email** - Send emails (custom implementations)
- **SMS** - Send text messages (custom implementations)

**Example:**
```yaml
reminder_agent:
  toolsets:
    - notification-server
  instructions:
    - |
        Send push notifications for:
        - Upcoming appointments
        - Important deadlines
        - Status updates
```

### External Services

Tools that integrate with third-party services:

- **Google Calendar** (`google-calendar`) - Calendar management
- **Payment Processors** - Handle transactions (custom implementations)
- **CRM Systems** - Customer relationship management (custom implementations)

### Specialized Tools

Domain-specific tools for particular use cases:

- **Product Catalogs** - E-commerce product data
- **Analytics** - Business intelligence and reporting
- **Calculation** - Mathematical and statistical operations

## Agent Types and Tools

Different agent types have different tool access patterns:

| Agent Type | Direct Tool Access | Tool Access via Sub-Agents |
|------------|-------------------|----------------------------|
| **LLM** | ✅ Yes | ✅ Yes (if sub-agents have tools) |
| **Parallel** | ❌ No | ✅ Yes (sub-agents can have tools) |
| **Loop** | ❌ No | ✅ Yes (sub-agent can have tools) |
| **Sequence** | ❌ No | ✅ Yes (sub-agents can have tools) |

### LLM Agent with Tools

Direct tool access:

```yaml
researcher:
  type: llm
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Use duckduckgo for research.
        Store findings in memory-server.
```

### Parallel Agent with Tools

Tools accessed by sub-agents:

```yaml
multi_researcher:
  type: parallel
  # No toolsets here
  sub_agents:
    - source_1
    - source_2
  
source_1:
  type: llm
  toolsets:
    - duckduckgo    # Tool access here
```

### Loop Agent with Tools

Tool accessed by iterative sub-agent:

```yaml
content_refiner:
  type: loop
  max_iterations: 3
  # No toolsets here
  sub_agents:
    - refiner

refiner:
  type: llm
  toolsets:
    - memory-server    # Tool access here
```

### Sequence Agent with Tools

Tools accessed by pipeline stages:

```yaml
pipeline:
  type: sequence
  # No toolsets here
  sub_agents:
    - research_stage
    - write_stage

research_stage:
  type: llm
  toolsets:
    - duckduckgo       # Tool access in stage 1

write_stage:
  type: llm
  toolsets:
    - memory-server    # Tool access in stage 2
```

## Tool Usage Patterns

### Research Pattern

Agent searches web and stores findings:

```yaml
researcher:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Research workflow:
        1. Search web using duckduckgo
        2. Analyze and summarize findings
        3. Store key insights in memory-server with descriptive keys
        4. Present summary to user
```

### Personalization Pattern

Agent retrieves user preferences and customizes behavior:

```yaml
assistant:
  toolsets:
    - memory-server
  instructions:
    - |
        Personalization:
        1. Check memory-server for user preferences (key: "user_preferences")
        2. Apply preferences to recommendations
        3. Update preferences based on user feedback
        4. Store updated preferences back to memory-server
```

### Notification Pattern

Agent sends alerts based on triggers:

```yaml
reminder_agent:
  toolsets:
    - google-calendar
    - notification-server
  instructions:
    - |
        Reminder workflow:
        1. Check google-calendar for upcoming events
        2. For events within 1 hour:
           - Send notification via notification-server
           - Include event details and location
        3. Mark notification as sent
```

### Multi-Tool Pattern

Agent coordinates multiple tools:

```yaml
travel_assistant:
  toolsets:
    - weather-tool
    - google-calendar
    - memory-server
  instructions:
    - |
        Travel planning:
        1. Check google-calendar for trip dates
        2. Get weather-tool forecast for destination
        3. Retrieve user preferences from memory-server
        4. Provide customized recommendations
        5. Store itinerary in memory-server
```

## Tool Selection Guidelines

### Minimal Tool Principle

Only include tools an agent actually needs:

**Good - Focused tools:**
```yaml
weather_agent:
  toolsets:
    - weather-tool    # Only tool needed
```

**Poor - Unnecessary tools:**
```yaml
weather_agent:
  toolsets:
    - weather-tool
    - duckduckgo          # Not needed
    - memory-server       # Not needed
    - google-calendar     # Not needed
```

### Tool Expertise

Match tools to agent specialization:

```yaml
# Research specialist
researcher:
  toolsets:
    - duckduckgo
    - memory-server

# Scheduling specialist
scheduler:
  toolsets:
    - google-calendar
    - notification-server

# Data analyst (no external tools needed)
analyst:
  # No toolsets - works with provided data
```

### Tool Combinations

Common tool combinations:

**Research & Storage:**
```yaml
toolsets:
  - duckduckgo      # Gather information
  - memory-server   # Store findings
```

**Scheduling & Notifications:**
```yaml
toolsets:
  - google-calendar      # Event management
  - notification-server  # Alerts
```

**Context & Personalization:**
```yaml
toolsets:
  - memory-server   # User preferences
  - weather-tool    # Contextual data
```

## Tool Instructions

### Clear Tool Guidance

Specify when and how to use each tool:

```yaml
instructions:
  - |
      Tool usage guidelines:
      
      duckduckgo:
      - Use for current information not in training data
      - Search for facts, statistics, recent news
      - Verify claims across multiple sources
      
      memory-server:
      - Store user preferences with key "user_prefs"
      - Store conversation context with key "context_{date}"
      - Retrieve before making recommendations
      
      notification-server:
      - Send for important updates only
      - Keep messages concise (under 100 chars)
      - Include actionable information
```

### Tool Error Handling

Guide agents on tool failures:

```yaml
instructions:
  - |
      If tool calls fail:
      
      duckduckgo fails:
      - Inform user search is unavailable
      - Use training data knowledge where appropriate
      - Caveat information may not be current
      
      memory-server fails:
      - Continue without stored preferences
      - Ask user for preferences directly
      - Don't block on storage errors
      
      notification-server fails:
      - Log the error
      - Present information directly in chat
      - Retry notification once
```

### Tool Result Interpretation

Guide agents on using tool results:

```yaml
instructions:
  - |
      Interpreting tool results:
      
      duckduckgo results:
      - Synthesize information from multiple sources
      - Cite sources when making claims
      - Note conflicting information
      - Assess source credibility
      
      memory-server results:
      - Handle missing keys gracefully
      - Validate stored data format
      - Check data freshness (timestamps)
      - Update stale data
```

## Security and Permissions

### Tool Access Control

Tools respect user permissions:

- ✅ Users control which tools can access their data
- ✅ Tools require authentication for sensitive operations
- ✅ System enforces rate limits and quotas
- ✅ Audit logs track tool usage

### Data Privacy

Tool data handling:

- **Memory Server**: User data is isolated per user
- **Notifications**: Only sent to authenticated user
- **Calendar**: Requires OAuth consent
- **Web Search**: No personal data in queries

### Best Practices

**Secure tool usage:**
```yaml
instructions:
  - |
      Privacy guidelines:
      - Don't store sensitive data (passwords, payment info)
      - Don't include personal data in web searches
      - Don't send sensitive info via notifications
      - Request minimal permissions needed
```

## Tool Availability

### Built-in Tools

Available in all MyAgents deployments:
- `memory-server` - Built-in data persistence
- `notification-server` - Built-in notifications

### Standard Tools

Configured per deployment:
- `duckduckgo` - Web search
- `weather-tool` - Weather data
- `google-calendar` - Calendar integration

### Administrator-Deployed Tools

Your organization's administrator may deploy additional tools for specific business needs:
- Product catalog integrations
- Internal API access
- Custom integrations
- Domain-specific services

Check with your administrator for available tools.

## Performance Considerations

### Tool Call Latency

Tool calls add processing time:

| Tool | Typical Latency |
|------|----------------|
| `memory-server` | 10-50ms |
| `duckduckgo` | 500-2000ms |
| `weather-tool` | 200-800ms |
| `google-calendar` | 300-1000ms |
| `notification-server` | 50-200ms |

### Optimization Tips

**Parallel tool calls:**
Agents can call multiple tools concurrently when results don't depend on each other.

**Caching:**
Store frequently-accessed data in memory-server to reduce repeated tool calls.

**Lazy loading:**
Only call tools when absolutely needed, not preemptively.

**Example - Optimized:**
```yaml
instructions:
  - |
      Optimization strategy:
      1. Check memory-server cache first
      2. If cache miss or stale, call duckduckgo
      3. Update cache with fresh results
      4. Use cached data for subsequent requests
```

## Tool Limitations

### Rate Limits

Most tools have rate limits:
- Web search: Limited queries per minute
- Calendar: API quota limits
- Notifications: Send frequency limits

### Data Freshness

Tool data has varying freshness:
- Web search: Minutes to hours old
- Weather: Updated hourly
- Calendar: Real-time
- Memory: Depends on last update

### Availability

Tools may be temporarily unavailable:
- Network issues
- API outages
- Rate limit exceeded
- Authentication failures

**Design for resilience:**
```yaml
instructions:
  - |
      Graceful degradation:
      - Always have a fallback if tool unavailable
      - Don't block user request on tool failure
      - Communicate limitations to user
      - Retry with exponential backoff
```

## Next Steps

- **[Available Tools](available-tools.md)** - Complete tool catalog with examples
- **[Using Tools](using-tools.md)** - Best practices and patterns
- **[Agent Examples](../agents/examples.md)** - Teams with tool integration

---

**Related:**  
[Getting Started](../getting-started.md) | [Creating Teams](../agents/creating-agents.md) | [Configuration](../agents/configuration.md)
