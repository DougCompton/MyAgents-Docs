# Available Tools

This reference lists all MCP tools available in the MyAgents platform.

## Quick Reference

| Tool | Category | Description | Authentication |
|------|----------|-------------|----------------|
| [duckduckgo](#duckduckgo) | Search | Web search engine | None |
| [weather-tool](#weather-tool) | Information | Weather data | None |
| [memory-server](#memory-server) | Storage | Persistent data storage | None |
| [notification-server](#notification-server) | Communication | Push notifications | None |
| [google-calendar](#google-calendar) | Integration | Calendar management | OAuth Required |
| [switch_agent](#switch_agent) | System | Switch to another agent | Built-in |
| [transfer_task](#transfer_task) | System | Delegate to sub-persona | Built-in |

## System Tools

These tools are automatically available and don't need to be specified in `toolsets`.

### switch_agent

**Purpose:** Switch from current agent to another specialized agent

**Category:** System / Built-in

**Availability:** Automatically available to all agents

**When to use:**
- Current agent cannot handle the user's request
- User's request is better suited for another specialized agent
- Need to transition workflow to different domain expert

**Usage in instructions:**
```json
{
  "content": "If the user asks about topics outside your expertise, use the switch_agent tool to transfer them to an appropriate specialist. For example, weather questions → weather agent, shopping assistance → shopping agent."
}
```

**Parameters:**
- `agent_id` (string): ID of target agent
- `reason` (string): Explanation of why switching

**Notes:**
- Only available to switchboard and coordinator agents
- Preserves conversation context
- User is informed of the switch

---

### transfer_task

**Purpose:** Delegate a task to a sub-persona within the same agent

**Category:** System / Built-in

**Availability:** Automatically available to coordinator personas

**When to use:**
- Multi-step workflows requiring specialization
- Sequential processing (research → write → edit)
- Parallel task decomposition

**Usage in instructions:**
```json
{
  "content": "To delegate tasks:\n1. Identify which sub-persona should handle the task\n2. Use transfer_task with clear task description\n3. Wait for sub-persona to complete\n4. Process and present results\n\nAvailable sub-personas: researcher, writer, editor"
}
```

**Parameters:**
- `target_persona` (string): Name of sub-persona
- `task_description` (string): What the sub-persona should do

**Notes:**
- Only available to personas with defined `subAgents`
- Task context is preserved
- Results return to calling persona

---

## Search & Information Tools

### duckduckgo

**Purpose:** Web search for current information and facts

**Category:** Search / Information Retrieval

**Authentication:** None required

**Toolset name:** `"duckduckgo"`

**Capabilities:**
- Search web content
- Find current information
- Retrieve factual data
- Discover relevant sources

**Best for:**
- Current events and news
- Product research and comparisons
- Fact-checking and verification
- Finding documentation and resources

**Usage example:**
```json
{
  "name": "researcher",
  "description": "Researches topics using web search",
  "instructions": [
    {
      "content": "When users request information:\n1. Use web search to find current, accurate data\n2. Cross-reference multiple sources\n3. Always cite sources with URLs\n4. Note publication dates for time-sensitive information\n5. Verify facts before presenting"
    }
  ],
  "toolsets": ["duckduckgo"]
}
```

**Rate limits:** Standard web search rate limits apply

**Notes:**
- Returns text-based search results
- No image or video search
- Respects privacy (no tracking)
- May have regional variations

---

### weather-tool

**Purpose:** Get current weather conditions and forecasts

**Category:** Information / Weather

**Authentication:** None required

**Toolset name:** `"weather-tool"`

**Capabilities:**
- Current weather conditions
- Temperature, humidity, wind
- Weather forecasts
- Severe weather alerts (US)

**Best for:**
- Weather inquiries
- Travel planning
- Event scheduling
- Outdoor activity planning

**Usage example:**
```json
{
  "name": "weather_assistant",
  "description": "Provides weather information",
  "instructions": [
    {
      "content": "Provide weather information clearly:\n- Current conditions: temperature, conditions, wind\n- Forecast: summarize next few days\n- Recommendations: appropriate for conditions\n- Alerts: highlight any severe weather warnings\n\nAlways specify location and time for context."
    }
  ],
  "toolsets": ["weather-tool"]
}
```

**Coverage:** United States (National Weather Service data)

**Rate limits:** Reasonable usage limits apply

**Notes:**
- US-focused weather data
- Updates every hour
- Includes severe weather alerts

---

## Storage & Data Tools

### memory-server

**Purpose:** Store and retrieve persistent data across sessions

**Category:** Storage / Database

**Authentication:** None required (user-scoped)

**Toolset name:** `"memory-server"`

**Capabilities:**
- Store user data persistently
- Retrieve stored information
- Update existing data
- Delete outdated information
- Search stored memories
- Category-based organization

**Best for:**
- User preferences and settings
- Shopping lists and wishlists
- Personal notes and reminders
- Historical data and patterns
- Session continuity

**Usage example:**
```json
{
  "name": "preference_manager",
  "description": "Manages user preferences and history",
  "instructions": [
    {
      "content": "Store user preferences for personalization:\n\n**Storing:**\n- Save preferences with descriptive categories\n- Use consistent naming conventions\n- Include context for future reference\n\n**Retrieving:**\n- Search memories by category or keywords\n- Combine multiple stored preferences\n- Note when preferences might be outdated\n\n**Privacy:**\n- Only store information user explicitly wants saved\n- Respect deletion requests immediately\n- Don't share personal data across users"
    }
  ],
  "toolsets": ["memory-server"]
}
```

**Available operations:**
- `create` - Store new memory
- `read` - Retrieve by ID
- `update` - Modify existing memory
- `delete` - Remove memory
- `list` - List all memories (with filters)
- `search` - Search by keywords

**Data structure:**
```json
{
  "id": "generated-uuid",
  "userId": "user-id",
  "agentId": "agent-id",
  "content": "The stored information",
  "category": "preferences",
  "createdAt": "2025-10-25T10:30:00Z"
}
```

**Storage scope:**
- User-scoped: Each user has isolated data
- Agent-scoped: Memories tied to specific agent
- Persistent: Survives system restarts

**Notes:**
- DynamoDB-backed for reliability
- Automatic user isolation
- Supports categories for organization
- Full-text search capabilities

---

## Communication Tools

### notification-server

**Purpose:** Send push notifications to users

**Category:** Communication / Alerts

**Authentication:** None required (user-scoped)

**Toolset name:** `"notification-server"`

**Capabilities:**
- Send push notifications
- Schedule future notifications
- Manage notification preferences
- Track delivery status

**Best for:**
- Reminders and alerts
- Task completion notifications
- Time-sensitive updates
- Event notifications

**Usage example:**
```json
{
  "name": "reminder_agent",
  "description": "Creates and manages reminders",
  "instructions": [
    {
      "content": "Manage user reminders:\n\n**Creating reminders:**\n- Confirm time and content with user\n- Use clear, actionable notification text\n- Respect user's timezone\n\n**Notification best practices:**\n- Keep messages concise (under 100 characters)\n- Include actionable information\n- Avoid excessive notifications\n- Respect quiet hours (10 PM - 8 AM)\n\n**Follow-up:**\n- Confirm notification was scheduled\n- Provide way to cancel or modify\n- Note when notification will be sent"
    }
  ],
  "toolsets": ["notification-server", "memory-server"]
}
```

**Available operations:**
- `send` - Send immediate notification
- `schedule` - Schedule future notification
- `cancel` - Cancel scheduled notification
- `listScheduled` - List upcoming notifications

**Notification structure:**
```json
{
  "title": "Reminder",
  "message": "Your scheduled event starts in 30 minutes",
  "scheduledFor": "2025-10-25T14:30:00Z"
}
```

**Delivery:**
- Push notifications to mobile devices
- Requires user to have push token registered
- Respects user notification preferences

**Notes:**
- User must enable push notifications
- Timezone-aware scheduling
- Delivery confirmation available

---

## Integration Tools

### google-calendar

**Purpose:** Access and manage Google Calendar events

**Category:** Integration / Calendar

**Authentication:** OAuth 2.0 required

**Toolset name:** `"google-calendar"`

**Capabilities:**
- View calendar events
- Create new events
- Update existing events
- Delete events
- Check availability
- Manage attendees

**Best for:**
- Meeting scheduling
- Calendar management
- Availability checking
- Event coordination

**Usage example:**
```json
{
  "name": "meeting_scheduler",
  "description": "Schedules meetings and manages calendar",
  "instructions": [
    {
      "content": "Help users manage their Google Calendar:\n\n**Viewing events:**\n- Check calendar before scheduling\n- Summarize upcoming events clearly\n- Note conflicts and double bookings\n\n**Scheduling:**\n- Confirm details before creating events\n- Suggest times based on availability\n- Add relevant attendees and locations\n- Set appropriate reminders\n\n**Modifications:**\n- Always confirm before changing events\n- Check with attendees for group events\n- Update all participants on changes"
    }
  ],
  "toolsets": ["google-calendar"]
}
```

**Available operations:**
- `listEvents` - List calendar events
- `createEvent` - Create new event
- `updateEvent` - Modify existing event
- `deleteEvent` - Remove event
- `findAvailability` - Check free time slots

**Authentication flow:**
1. User grants calendar access (OAuth)
2. System stores access token securely
3. Agent uses token for calendar operations
4. User can revoke access anytime

**Permissions required:**
- `https://www.googleapis.com/auth/calendar` - Full calendar access

**Notes:**
- Requires user to connect Google account
- Respects calendar sharing settings
- Works with multiple calendars
- Supports recurring events

---

## Tool Selection Guide

### By Use Case

**Information Gathering:**
- General web research → `duckduckgo`
- Weather information → `weather-tool`

**Data Management:**
- User preferences → `memory-server`
- Personal data → `memory-server`

**Communication:**
- Reminders → `notification-server`
- Alerts → `notification-server`

**Scheduling:**
- Calendar events → `google-calendar`
- Availability checking → `google-calendar`

**Workflow Management:**
- Agent switching → `switch_agent` (automatic)
- Task delegation → `transfer_task` (automatic)

### By Agent Type

**Research Agents:**
```json
"toolsets": ["duckduckgo"]
```

**Personal Assistants:**
```json
"toolsets": ["memory-server", "notification-server", "google-calendar"]
```

**Shopping Assistants:**
```json
"toolsets": ["duckduckgo", "memory-server"]
```

**Content Creation:**
```json
"toolsets": ["duckduckgo"]
```

**Coordinators:**
```json
// Usually no tools - just delegates via transfer_task
"toolsets": []
```

## Advanced: Tool Exclusions

Conditionally disable tools based on settings:

```json
{
  "name": "data_manager",
  "toolsets": [
    {
      "name": "memory-server",
      "exclude": [
        {
          "tool": "delete",
          "if": "setting.readOnlyMode == true"
        },
        {
          "tool": "update",
          "if": "setting.readOnlyMode == true"
        }
      ]
    }
  ]
}
```

## Troubleshooting

### Tool Not Available

**Error:** "Unknown or disabled MCP toolset"

**Checklist:**
1. Verify toolset name spelling (case-sensitive)
2. Check tool is registered in `mcp.servers.json`
3. Ensure server is running (for external tools)
4. Review system logs for errors

### Authentication Failures

**Error:** "Authentication required" or "Permission denied"

**For OAuth tools (google-calendar):**
1. User must complete OAuth flow
2. Check token hasn't expired
3. Verify required scopes are granted
4. Re-authenticate if needed

### Tool Call Failures

**Error:** Tool operation fails or returns error

**Debugging steps:**
1. Check tool parameters are correct format
2. Verify network connectivity (external tools)
3. Review rate limits and quotas
4. Check tool-specific logs
5. Test tool independently

## Next Steps

- **[Using Tools](using-tools.md)** - Best practices for tool integration
- **[Creating Tools](creating-tools.md)** - Build custom MCP servers
- **[Agent Examples](../agents/examples.md)** - See tools in action
- **[Tool Security](using-tools.md#security)** - Security considerations

---

**Related:**  
[Tools Overview](overview.md) | [Using Tools](using-tools.md) | [Creating Agents](../agents/creating-agents.md)

