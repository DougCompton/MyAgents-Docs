# Using Tools

Best practices and patterns for integrating MCP tools into your agents.

## Tool Integration Patterns

### Pattern 1: Research and Information Gathering

Use web search tools to gather current information.

```json
{
  "name": "researcher",
  "description": "Researches topics using web search",
  "instructions": [
    {
      "content": "When researching topics:\n\n**Search Strategy:**\n1. Use specific, targeted search queries\n2. Search multiple times for comprehensive coverage\n3. Cross-reference information from multiple sources\n4. Note publication dates for time-sensitive info\n\n**Source Quality:**\n- Prefer authoritative sources (.edu, .gov, established media)\n- Check author credentials when available\n- Note any potential bias\n- Verify facts across sources\n\n**Citation:**\n- Always include source URLs\n- Note publication dates\n- Cite key quotes with attribution\n\n**Present findings clearly with proper citations**"
    }
  ],
  "toolsets": ["duckduckgo"]
}
```

**Best for:**
- Research agents
- Fact-checking
- Current events
- Product research

---

### Pattern 2: Persistent Data Storage

Use memory-server to store and retrieve user data across sessions.

```json
{
  "name": "preference_manager",
  "description": "Manages user preferences and shopping history",
  "instructions": [
    {
      "content": "Manage user preferences:\n\n**Storing Data:**\n```\n- Save after user expresses preference\n- Use descriptive categories (preferences, history, settings)\n- Include context for future reference\n- Confirm save with user\n```\n\n**Retrieving Data:**\n```\n- Check for existing preferences at start of session\n- Use categories to organize retrieval\n- Combine multiple preferences intelligently\n- Note when preferences might be outdated\n```\n\n**Privacy:**\n```\n- Only store what user explicitly wants saved\n- Explain what's being stored\n- Honor deletion requests immediately\n- Don't share data across users\n```"
    }
  ],
  "toolsets": ["memory-server"]
}
```

**Storage categories:**
```json
{
  "category": "user-preferences",
  "content": "Prefers organic products, budget-conscious, vegetarian diet"
}
```

```json
{
  "category": "shopping-history",
  "content": "Frequently purchases: almond milk, quinoa, fresh vegetables"
}
```

**Best for:**
- Personalization
- User preferences
- Shopping lists
- Historical data

---

### Pattern 3: Scheduled Notifications

Use notification-server to send timely reminders.

```json
{
  "name": "reminder_agent",
  "description": "Creates and manages reminders",
  "instructions": [
    {
      "content": "Manage reminders:\n\n**Creating Reminders:**\n```\n1. Confirm time and timezone with user\n2. Use clear, actionable notification text\n3. Keep messages under 100 characters\n4. Include essential info only\n```\n\n**Timing:**\n```\n- Respect user's quiet hours (10 PM - 8 AM)\n- Send work reminders during business hours\n- Consider lead time (notify 30min before events)\n```\n\n**Follow-up:**\n```\n- Confirm notification was scheduled\n- Provide way to cancel/modify\n- Store reminder in memory for reference\n```"
    }
  ],
  "toolsets": ["notification-server", "memory-server"]
}
```

**Best for:**
- Reminder systems
- Event notifications
- Task alerts
- Time-sensitive updates

---

### Pattern 4: Calendar Integration

Use google-calendar for scheduling and availability checking.

```json
{
  "name": "meeting_scheduler",
  "description": "Schedules meetings and checks availability",
  "instructions": [
    {
      "content": "Manage calendar:\n\n**Scheduling Meetings:**\n```\n1. Check current calendar for conflicts\n2. Suggest 2-3 available time slots\n3. Consider meeting duration and buffer time\n4. Add location and video link if relevant\n5. Confirm before creating event\n```\n\n**Finding Availability:**\n```\n- Look for gaps between existing events\n- Consider minimum meeting duration\n- Respect working hours preferences\n- Flag potential conflicts\n```\n\n**Modifications:**\n```\n- Always confirm before changing events\n- Notify attendees of changes\n- Check for cascading conflicts\n```"
    }
  ],
  "toolsets": ["google-calendar"]
}
```

**Best for:**
- Meeting scheduling
- Calendar management
- Availability checking
- Event coordination

---

## Multi-Tool Patterns

### Pattern 5: Research + Storage

Combine web search with persistent storage for personalized research.

```json
{
  "name": "personalized_researcher",
  "description": "Researches topics and remembers user interests",
  "instructions": [
    {
      "content": "Provide personalized research:\n\n**Workflow:**\n1. Check memory for user interests and past research\n2. Use web search for current information\n3. Filter/prioritize based on user preferences\n4. Save new insights to memory\n5. Build on previous research\n\n**Example:**\n- User interested in 'sustainable fashion'\n- Search for latest sustainable brands\n- Filter by previously noted preferences (budget, style)\n- Save new brands to memory\n- Reference past findings in new research"
    }
  ],
  "toolsets": ["duckduckgo", "memory-server"]
}
```

---

### Pattern 6: Calendar + Notifications

Combine calendar with notifications for complete event management.

```json
{
  "name": "event_manager",
  "description": "Manages events with reminders",
  "instructions": [
    {
      "content": "Manage events end-to-end:\n\n**When creating events:**\n1. Add to calendar\n2. Schedule notification 30min before\n3. Schedule notification 1 day before (for important events)\n4. Confirm both were created\n\n**For recurring events:**\n- Set up calendar recurrence\n- Schedule notification for each occurrence\n- Store event details in memory\n\n**For event changes:**\n- Update calendar\n- Cancel old notifications\n- Create new notifications\n- Inform user of all changes"
    }
  ],
  "toolsets": ["google-calendar", "notification-server", "memory-server"]
}
```

---

## Tool Configuration

### Conditional Tool Access

Restrict tools based on agent mode or user permissions.

```json
{
  "name": "data_manager",
  "description": "Manages data with role-based access",
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

**Use cases:**
- Read-only mode for viewers
- Limited access for trial users
- Safety mode for testing
- Compliance requirements

---

## Security Best Practices

### 1. Data Privacy

**DO:**
- Only store necessary information
- Use user-scoped storage (memory-server handles this automatically)
- Explain what data is being saved
- Honor deletion requests promptly

**DON'T:**
- Store sensitive information (passwords, credit cards)
- Share user data across accounts
- Keep data indefinitely without purpose

### 2. Tool Error Handling

```json
{
  "content": "Handle tool errors gracefully:\n\n**Network Failures:**\n- Inform user of connection issue\n- Suggest retry after brief wait\n- Provide cached/alternative info if available\n\n**Permission Errors:**\n- Explain authentication need\n- Guide through OAuth flow\n- Respect declined permissions\n\n**Rate Limits:**\n- Explain temporary limitation\n- Provide estimated wait time\n- Suggest alternatives if available"
}
```

### 3. OAuth Tools

For tools requiring OAuth (google-calendar):

```json
{
  "content": "OAuth tool usage:\n\n**First Use:**\n1. Explain why permission is needed\n2. List specific permissions required\n3. Guide through authorization\n4. Confirm successful connection\n\n**Ongoing:**\n- Check token validity before operations\n- Handle re-authentication gracefully\n- Respect scope limitations\n\n**User Control:**\n- Explain how to revoke access\n- Don't request unnecessary permissions\n- Store tokens securely (handled by system)"
}
```

---

## Performance Optimization

### Minimize Tool Calls

**Inefficient:**
```
1. Search for "best laptops"
2. Search for "laptop reviews"
3. Search for "laptop prices"
4. Search for "where to buy laptops"
```

**Efficient:**
```
1. Search for "best laptops 2025 reviews prices"
2. Process comprehensive results
```

### Batch Operations

When using memory-server, batch related operations:

```json
{
  "content": "Store related preferences together:\n\n**Instead of:**\n- Save \"likes organic\" (separate call)\n- Save \"budget conscious\" (separate call)\n- Save \"vegetarian\" (separate call)\n\n**Do:**\n- Save \"Shopping preferences: organic, budget-conscious, vegetarian\" (single call)"
}
```

### Cache Results

For expensive operations, save results:

```json
{
  "content": "Cache search results:\n\n1. Search web for product info\n2. Save results to memory-server\n3. For follow-up questions, check memory first\n4. Only search again if info is stale (>1 day old)"
}
```

---

## Testing Tools

### Development Testing

Test each tool independently:

```json
{
  "name": "tool_tester",
  "instructions": [
    {
      "content": "Test tools systematically:\n\n**Web Search:**\n- Test with various query types\n- Verify results are current\n- Check error handling\n\n**Memory:**\n- Test save/retrieve cycle\n- Verify data isolation\n- Test deletion\n\n**Notifications:**\n- Test immediate send\n- Test scheduled notifications\n- Verify delivery"
    }
  ],
  "toolsets": ["duckduckgo", "memory-server", "notification-server"]
}
```

### Error Scenarios

Test tool failure handling:

- Network disconnection
- Invalid parameters
- Rate limiting
- Permission denied
- Service unavailable

---

## Tool Combination Guidelines

### Compatible Combinations

**Research + Storage:**
```
duckduckgo + memory-server
```
Store research findings for future reference

**Calendar + Notifications:**
```
google-calendar + notification-server
```
Complete event management with reminders

**Search + All Storage:**
```
duckduckgo + memory-server + notification-server
```
Research, save findings, set follow-up reminders

### Avoid Redundancy

**Don't combine tools that duplicate functionality:**
- Multiple search tools (stick to one)
- Multiple storage tools (memory-server is sufficient)

---

## Next Steps

- **[Available Tools](available-tools.md)** - Complete tool catalog
- **[Creating Tools](creating-tools.md)** - Build custom MCP servers
- **[Agent Examples](../agents/examples.md)** - See tools in action

---

**Related:**  
[Tools Overview](overview.md) | [Available Tools](available-tools.md) | [Creating Agents](../agents/creating-agents.md)

