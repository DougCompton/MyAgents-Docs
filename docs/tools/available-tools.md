# Available Tools

Complete catalog of tools available for agent integration.

## Core Tools

### DuckDuckGo Search

**Toolset ID:** `duckduckgo`  
**Category:** Information Gathering  
**Authentication:** None required

Web search capabilities for finding current information.

#### Capabilities

- 🔍 General web search
- 📰 Recent news and articles
- 📚 Reference information
- 🎯 Targeted queries
- 🌐 Multi-language support

#### Configuration

```yaml
agents:
  researcher:
    toolsets:
      - duckduckgo
    instructions:
      - |
          Use duckduckgo to search for current information.
          Synthesize results from multiple sources.
          Always cite sources when presenting findings.
```

#### Usage Examples

**Research Assistant:**
```yaml
research_agent:
  toolsets:
    - duckduckgo
  instructions:
    - |
        Research workflow:
        1. Search duckduckgo for topic
        2. Review top 5-10 results
        3. Cross-reference facts
        4. Identify authoritative sources
        5. Present synthesized findings with citations
```

**Fact Checker:**
```yaml
fact_checker:
  toolsets:
    - duckduckgo
  instructions:
    - |
        Fact checking:
        1. Search for claim using duckduckgo
        2. Find multiple independent sources
        3. Assess source credibility
        4. Report: Confirmed / Disputed / Unverified
```

**News Researcher:**
```yaml
news_researcher:
  toolsets:
    - duckduckgo
  instructions:
    - |
        Find recent news:
        - Add "news" or date filters to queries
        - Focus on reputable news sources
        - Check multiple outlets for coverage
        - Note publication dates
```

#### Best Practices

**Effective Queries:**
- ✅ Be specific and targeted
- ✅ Use quotes for exact phrases
- ✅ Include relevant keywords
- ✅ Filter by date when needed

**Result Processing:**
- ✅ Review multiple results
- ✅ Cross-reference information
- ✅ Cite sources
- ✅ Note conflicting information

**Limitations:**
- ⚠️ Results may be minutes to hours old
- ⚠️ Source quality varies
- ⚠️ Rate limits apply
- ⚠️ Cannot access paywalled content

---

### Memory Server

**Toolset ID:** `memory-server`  
**Category:** Data Persistence  
**Authentication:** Automatic (user-scoped)

Persistent key-value storage for user data, preferences, and context.

#### Capabilities

- 💾 Store user preferences
- 📝 Save conversation context
- 🔖 Bookmark important information
- 📊 Track user data over time
- 🔒 User-isolated storage (data privacy)

#### Configuration

```yaml
agents:
  personalized_assistant:
    toolsets:
      - memory-server
    instructions:
      - |
          Use memory-server to provide personalized experience:
          - Retrieve user preferences at start of conversation
          - Store new preferences as user provides them
          - Remember important context between sessions
```

#### Key Structure

Use descriptive, hierarchical keys:

```
user_preferences          # General user preferences
dietary_restrictions      # Specific preference type
shopping_history_{date}   # Time-stamped data
conversation_context      # Session context
favorites_products        # Collections
settings_{category}       # Configuration
```

#### Usage Examples

**User Preferences:**
```yaml
shopping_assistant:
  toolsets:
    - memory-server
  instructions:
    - |
        Preference management:
        
        On first interaction:
        - Check memory-server for key "user_preferences"
        - If not found, ask user about preferences
        - Store preferences with key "user_preferences"
        
        For each session:
        - Load preferences at start
        - Apply to recommendations
        - Update based on user feedback
        - Save updates back to memory-server
```

**Conversation Context:**
```yaml
content_assistant:
  toolsets:
    - memory-server
  instructions:
    - |
        Context persistence:
        
        At conversation start:
        - Retrieve key "current_project" for ongoing work
        
        During conversation:
        - Update "current_project" with progress
        - Store "recent_topics" for quick reference
        
        At conversation end:
        - Save final state to "current_project"
        - Store summary in "session_history_{timestamp}"
```

**Learning Agent:**
```yaml
learning_assistant:
  toolsets:
    - memory-server
  instructions:
    - |
        Learn from interactions:
        
        Store with structured keys:
        - "user_skill_level_{topic}": beginner/intermediate/advanced
        - "user_interests": Array of interest areas
        - "completed_tutorials": Track progress
        - "feedback_history": Learn from corrections
        
        Adapt teaching based on stored data.
```

**Shopping History:**
```yaml
product_recommender:
  toolsets:
    - memory-server
  instructions:
    - |
        Track shopping behavior:
        
        Store:
        - "viewed_products": Recently viewed items
        - "purchased_products": Purchase history
        - "favorite_brands": Preferred brands
        - "price_sensitivity": Budget preferences
        
        Use history to:
        - Avoid recommending recently viewed items
        - Suggest complementary products
        - Respect brand preferences
        - Match budget expectations
```

#### Best Practices

**Key Naming:**
- ✅ Use descriptive names: `dietary_restrictions` not `dr`
- ✅ Use snake_case: `user_preferences`
- ✅ Include timestamps when needed: `session_{timestamp}`
- ✅ Namespace related data: `settings_*`, `history_*`

**Data Structure:**
- ✅ Store as JSON for complex data
- ✅ Include metadata (timestamps, versions)
- ✅ Keep values reasonably sized (< 10KB)
- ✅ Use separate keys for separate concerns

**Data Management:**
- ✅ Check for existence before retrieval
- ✅ Handle missing keys gracefully
- ✅ Validate retrieved data format
- ✅ Update stale data
- ✅ Clean up old data periodically

**Privacy:**
- ❌ Never store sensitive data (passwords, payment info)
- ❌ Don't store personally identifiable information unnecessarily
- ✅ Store only what's needed for functionality
- ✅ Respect user data deletion requests

---

### Notification Server

**Toolset ID:** `notification-server`  
**Category:** Communication  
**Authentication:** Automatic (user-scoped)

Send push notifications to users.

#### Capabilities

- 📬 Push notifications to user devices
- ⏰ Reminder and alert notifications
- 📢 Status update notifications
- 🔔 Real-time alerts
- 📱 Cross-platform delivery

#### Configuration

```yaml
agents:
  reminder_agent:
    toolsets:
      - notification-server
    instructions:
      - |
          Send notifications for:
          - Upcoming appointments (15 min before)
          - Task deadlines (1 hour before)
          - Important status changes
          
          Keep messages concise and actionable.
```

#### Usage Examples

**Reminder Service:**
```yaml
reminder_agent:
  toolsets:
    - google-calendar
    - notification-server
  instructions:
    - |
        Reminder workflow:
        
        Every 15 minutes:
        1. Check google-calendar for events in next hour
        2. For each upcoming event:
           - Send notification-server alert
           - Include: event title, time, location
           - Add action: "View Details"
        3. Mark notification as sent (avoid duplicates)
```

**Status Updates:**
```yaml
process_monitor:
  toolsets:
    - notification-server
    - memory-server
  instructions:
    - |
        Process monitoring:
        
        When long-running task completes:
        1. Send notification-server update:
           - "Your report is ready"
           - "Click to view results"
        2. Store completion in memory-server
        3. Include summary in notification
```

**Alert System:**
```yaml
alert_agent:
  toolsets:
    - notification-server
  instructions:
    - |
        Alert for critical events:
        
        Severity levels:
        - High: Immediate notification with priority flag
        - Medium: Standard notification
        - Low: Non-urgent notification
        
        Include:
        - Clear description of event
        - Impact or urgency
        - Recommended action
        - Link to details
```

**Smart Notifications:**
```yaml
smart_notifier:
  toolsets:
    - notification-server
    - memory-server
  instructions:
    - |
        Intelligent notification:
        
        Check memory-server for:
        - User notification preferences
        - Quiet hours setting
        - Notification frequency limits
        
        Respect preferences:
        - Don't send during quiet hours
        - Batch low-priority notifications
        - Limit frequency (max 5/hour)
        
        Store notification history in memory-server.
```

#### Best Practices

**Message Content:**
- ✅ Keep under 100 characters
- ✅ Start with most important info
- ✅ Include clear action if needed
- ✅ Use friendly, human tone

**Timing:**
- ✅ Consider user timezone
- ✅ Respect quiet hours
- ✅ Don't spam (rate limit)
- ✅ Time-sensitive first

**User Experience:**
- ✅ Make notifications actionable
- ✅ Provide context
- ✅ Allow dismissal
- ✅ Don't interrupt unless urgent

**Examples:**

**Good:**
```
✅ "Meeting in 15 min: Design Review (Room 3B)"
✅ "Your report is ready. Tap to view."
✅ "⚠️ Server alert: High CPU usage detected"
```

**Poor:**
```
❌ "Notification" (not descriptive)
❌ "Hey there! So like, there's this thing that happened..." (too long/casual)
❌ Sending 20 notifications in 5 minutes (spam)
```

---

### Weather Tool

**Toolset ID:** `weather-tool`  
**Category:** Information Gathering  
**Authentication:** None required

Real-time weather data and forecasts.

#### Capabilities

- 🌡️ Current conditions
- 📅 Multi-day forecasts
- ⚠️ Severe weather alerts
- 🌍 Global coverage
- 📊 Detailed metrics (temp, humidity, wind, etc.)

#### Configuration

```yaml
agents:
  weather_assistant:
    toolsets:
      - weather-tool
    instructions:
      - |
          Provide weather information:
          - Current conditions with all details
          - Multi-day forecasts when asked
          - Alert users to severe weather
          - Format temperatures based on user preference
```

#### Usage Examples

**Weather Assistant:**
```yaml
weather_agent:
  toolsets:
    - weather-tool
  instructions:
    - |
        Weather information:
        
        For current conditions:
        - Temperature and "feels like"
        - Weather description (sunny, cloudy, etc.)
        - Humidity, wind speed, visibility
        - Precipitation if any
        
        For forecasts:
        - Daily high/low
        - Precipitation chance
        - General conditions
        - Day-by-day breakdown
        
        For severe weather:
        - Alert type and level
        - Affected areas
        - Duration
        - Safety recommendations
```

**Travel Planner:**
```yaml
travel_planner:
  toolsets:
    - weather-tool
    - memory-server
  instructions:
    - |
        Travel weather planning:
        
        For trip destination:
        1. Get weather-tool forecast for travel dates
        2. Analyze conditions:
           - Ideal weather: outdoor activities
           - Rainy: indoor alternatives
           - Extreme weather: warnings
        3. Provide packing recommendations
        4. Suggest activity adjustments
        5. Store itinerary in memory-server
```

**Activity Recommender:**
```yaml
activity_recommender:
  toolsets:
    - weather-tool
  instructions:
    - |
        Weather-based recommendations:
        
        Check weather-tool for today + weekend:
        
        Sunny & warm:
        - Outdoor activities
        - Parks, beaches
        - Sports
        
        Rainy:
        - Indoor venues
        - Museums, movies
        - Indoor sports
        
        Extreme weather:
        - Stay home suggestions
        - Safety-first recommendations
```

#### Best Practices

**Location Handling:**
- ✅ Support city names, ZIP codes, coordinates
- ✅ Confirm location if ambiguous
- ✅ Store preferred location in memory-server
- ✅ Handle location errors gracefully

**Temperature Units:**
- ✅ Ask user preference (F/C)
- ✅ Store preference in memory-server
- ✅ Convert consistently
- ✅ Display both if helpful

**Presentation:**
- ✅ Human-friendly descriptions
- ✅ Relevant details for context
- ✅ Highlight severe weather
- ✅ Include timestamps

---

### Google Calendar

**Toolset ID:** `google-calendar`  
**Category:** External Service  
**Authentication:** OAuth required

Calendar integration for event management.

#### Capabilities

- 📅 View calendar events
- ➕ Create new events
- ✏️ Update existing events
- 🗑️ Delete events
- 🔍 Search events
- 👥 Manage attendees
- ⏰ Set reminders

#### Configuration

```yaml
agents:
  scheduling_assistant:
    toolsets:
      - google-calendar
      - notification-server
    instructions:
      - |
          Manage calendar:
          - View upcoming events
          - Schedule new meetings
          - Find available time slots
          - Send reminders via notifications
```

#### Usage Examples

**Scheduling Assistant:**
```yaml
scheduler:
  toolsets:
    - google-calendar
    - notification-server
  instructions:
    - |
        Scheduling workflow:
        
        To schedule meeting:
        1. Query google-calendar for conflicts
        2. Find available time slots
        3. Propose options to user
        4. Create event in google-calendar
        5. Send confirmation via notification-server
        
        To find availability:
        1. Check google-calendar for date range
        2. Identify free blocks
        3. Consider preferences (morning/afternoon)
        4. Suggest optimal times
```

**Meeting Preparation:**
```yaml
meeting_prep:
  toolsets:
    - google-calendar
    - memory-server
    - notification-server
  instructions:
    - |
        Meeting preparation:
        
        15 minutes before meeting:
        1. Retrieve meeting from google-calendar
        2. Load related notes from memory-server
        3. Prepare agenda/context
        4. Send prep notification-server alert
        
        After meeting:
        1. Ask for notes
        2. Store in memory-server linked to event
        3. Set follow-up reminders if needed
```

**Calendar Analysis:**
```yaml
calendar_analyst:
  toolsets:
    - google-calendar
  instructions:
    - |
        Calendar insights:
        
        Analyze google-calendar events:
        - Meeting time by category
        - Free time availability
        - Busiest days/times
        - Recurring meeting load
        
        Provide:
        - Time management insights
        - Schedule optimization suggestions
        - Work-life balance assessment
```

#### Best Practices

**Event Creation:**
- ✅ Include all relevant details (title, time, location, description)
- ✅ Set appropriate reminders
- ✅ Add attendees when known
- ✅ Use clear, descriptive titles

**Time Handling:**
- ✅ Respect user timezone
- ✅ Confirm times before creating events
- ✅ Handle all-day events appropriately
- ✅ Consider duration (30min, 1hr defaults)

**Privacy:**
- ✅ Ask before reading calendar
- ✅ Only access needed date ranges
- ✅ Don't expose sensitive event details unnecessarily
- ✅ Respect calendar visibility settings

---

## Administrator-Deployed Tools

Your organization's administrator may deploy additional custom tools for specific business needs:

### Product Catalog Tool

**Example Configuration:**
```yaml
product_specialist:
  toolsets:
    - product-catalog
  instructions:
    - |
        Search product-catalog:
        - Filter by category, price range, availability
        - Sort by relevance, price, rating
        - Return top 5 matches
        - Include: name, price, rating, stock status
```

### CRM Integration

**Example Configuration:**
```yaml
sales_agent:
  toolsets:
    - crm-tool
  instructions:
    - |
        CRM operations:
        - Look up customer history
        - Update contact records
        - Log interactions
        - Track opportunities
```

### Analytics Tool

**Example Configuration:**
```yaml
analyst:
  toolsets:
    - analytics-tool
  instructions:
    - |
        Query analytics:
        - Retrieve metrics for date range
        - Generate reports
        - Calculate trends
        - Export data for visualization
```

## Tool Comparison

### By Use Case

| Use Case | Recommended Tools |
|----------|------------------|
| Research & Information | `duckduckgo`, `memory-server` |
| Personalization | `memory-server` |
| Scheduling & Time | `google-calendar`, `notification-server` |
| Weather & Location | `weather-tool`, `memory-server` |
| Alerts & Updates | `notification-server` |
| Data Persistence | `memory-server` |
| External API | Administrator-deployed tools |

### By Latency

| Tool | Latency | Use When |
|------|---------|----------|
| `memory-server` | Very Low (10-50ms) | Always acceptable |
| `notification-server` | Low (50-200ms) | Always acceptable |
| `weather-tool` | Medium (200-800ms) | Acceptable for most uses |
| `google-calendar` | Medium (300-1000ms) | Acceptable for most uses |
| `duckduckgo` | High (500-2000ms) | When current info needed |

### By Data Privacy

| Tool | Privacy Level | Data Handling |
|------|--------------|---------------|
| `memory-server` | High | User-isolated, encrypted |
| `notification-server` | High | User-scoped delivery |
| `google-calendar` | Medium | OAuth-protected, Google privacy policy |
| `weather-tool` | High | No personal data |
| `duckduckgo` | High | No personal data in queries |

## Tool Discovery

### Checking Available Tools

Your administrator can provide a list of available tools for your deployment.

### Requesting New Tools

If you need a tool that's not available:

1. Identify the specific capability needed
2. Check if existing tools can be combined
3. Request custom tool from administrator
4. Provide use case and requirements

### Testing Tools

Test tool availability:

```yaml
test_agent:
  toolsets:
    - tool-to-test
  instructions:
    - |
        Test tool functionality:
        - Verify tool responds
        - Check response format
        - Test error handling
        - Validate permissions
```

## Next Steps

- **[Using Tools](using-tools.md)** - Best practices and integration patterns
- **[Tools Overview](overview.md)** - Understanding tool architecture
- **[Agent Examples](../agents/examples.md)** - Teams with tool integration

---

**Related:**  
[Overview](overview.md) | [Using Tools](using-tools.md) | [Creating Teams](../agents/creating-agents.md)
