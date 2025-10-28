# Using Tools

Best practices and patterns for integrating tools into your agent teams.

## Getting Started with Tools

### Basic Tool Integration

The simplest tool integration:

```yaml
agents:
  weather_helper:
    type: llm
    toolsets:
      - weather-tool
    instructions:
      - |
          When users ask about weather, use the weather tool to get current 
          conditions or forecasts. Present the information clearly.
```

**That's it!** The agent automatically:
- Receives tool descriptions
- Knows when to use the tool
- Invokes the tool correctly
- Processes tool results
- Incorporates results into responses

### Adding Multiple Tools

```yaml
agents:
  research_assistant:
    type: llm
    toolsets:
      - duckduckgo       # Web search
      - memory-server    # Data storage
    instructions:
      - |
          Research topics using web search.
          Store important findings in memory for future reference.
```

## Tool Usage Patterns

### Research Pattern

Search, analyze, and store findings.

```yaml
researcher:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Research workflow:
        
        1. **Search Phase**
           - Use duckduckgo to search for topic
           - Gather information from top results
           - Cross-reference multiple sources
        
        2. **Analysis Phase**
           - Synthesize findings
           - Identify key insights
           - Note any conflicts or gaps
        
        3. **Storage Phase**
           - Store key findings in memory-server
           - Use descriptive keys: "research_{topic}_{date}"
           - Include source citations
        
        4. **Presentation Phase**
           - Present organized summary to user
           - Cite sources
           - Note confidence level
```

**Key benefits:**
- Systematic information gathering
- Persistent knowledge base
- Source traceability
- Reusable research

### Personalization Pattern

Retrieve and apply user preferences.

```yaml
personal_assistant:
  toolsets:
    - memory-server
  instructions:
    - |
        Personalization workflow:
        
        **Session Start:**
        1. Retrieve "user_preferences" from memory-server
        2. Load "conversation_context" if continuing work
        3. Apply preferences to behavior
        
        **During Conversation:**
        1. Note new preferences user expresses
        2. Update "user_preferences" immediately
        3. Apply new preferences to current and future responses
        
        **Session End:**
        1. Store "conversation_context" for continuity
        2. Save any preference updates
        3. Store "session_summary_{timestamp}"
        
        **Preference Types:**
        - Communication style (formal/casual)
        - Output format (detailed/concise)
        - Topics of interest
        - Dietary restrictions
        - Timezone and locale
```

**Key benefits:**
- Consistent experience across sessions
- Adaptive behavior
- User doesn't repeat preferences
- Context continuity

### Notification Pattern

Monitor and alert users.

```yaml
reminder_service:
  toolsets:
    - google-calendar
    - notification-server
    - memory-server
  instructions:
    - |
        Reminder workflow:
        
        **Monitoring Loop:**
        Every 15 minutes:
        1. Check google-calendar for upcoming events (next 60 min)
        2. Load "notified_events" from memory-server
        3. Filter out already-notified events
        
        **Notification Decision:**
        For each unnotified upcoming event:
        - 60 min before: Optional heads-up
        - 15 min before: Always notify
        - 5 min before: Urgent reminder
        
        **Send Notification:**
        1. Send via notification-server with:
           - Event title and time
           - Location (if any)
           - Action button: "View Details"
        2. Add event ID to "notified_events" in memory-server
        3. Log notification for tracking
        
        **Error Handling:**
        - If notification fails, retry once after 1 minute
        - If still fails, log error but don't block
```

**Key benefits:**
- Timely alerts
- No duplicate notifications
- Persistent tracking
- Graceful failures

### Orchestration Pattern

Coordinate multiple tools for complex workflows.

```yaml
travel_planner:
  toolsets:
    - google-calendar
    - weather-tool
    - duckduckgo
    - memory-server
  instructions:
    - |
        Travel planning workflow:
        
        **Phase 1: Information Gathering**
        1. Get trip dates from user or google-calendar
        2. Get destination weather forecast via weather-tool
        3. Research destination via duckduckgo:
           - Attractions and activities
           - Restaurants and hotels
           - Local transportation
           - Current events
        
        **Phase 2: Personalization**
        1. Load "travel_preferences" from memory-server:
           - Budget level
           - Activity preferences
           - Dietary restrictions
           - Accommodation preferences
        2. Filter recommendations by preferences
        
        **Phase 3: Itinerary Creation**
        1. Create day-by-day plan considering:
           - Weather forecast
           - User preferences
           - Travel time between locations
        2. Store itinerary in memory-server: "trip_{destination}_{date}"
        
        **Phase 4: Calendar Integration**
        1. Offer to add to google-calendar
        2. Create events for:
           - Flight times
           - Hotel check-in/out
           - Reserved activities
           - Restaurant reservations
        
        **Phase 5: Ongoing Support**
        1. Store plan in memory-server for later reference
        2. Offer to send reminders day-of
        3. Provide packing checklist based on weather
```

**Key benefits:**
- Comprehensive planning
- Personalized recommendations
- Integrated schedule
- Persistent itinerary

## Tool Instruction Patterns

### Explicit Tool Guidance

Specify exactly when and how to use each tool.

```yaml
instructions:
  - |
      Tool Usage Guidelines:
      
      **duckduckgo (Web Search)**
      When to use:
      - User asks about current events
      - Need facts not in training data
      - Research required
      
      How to use:
      - Formulate specific, targeted queries
      - Review top 5-10 results
      - Cross-reference information
      - Cite sources in response
      
      When NOT to use:
      - Information in training data
      - Personal opinions (no factual answer)
      - Already searched recently (check memory first)
      
      **memory-server (Data Persistence)**
      When to use:
      - Storing user preferences
      - Saving research findings
      - Preserving conversation context
      - Tracking user data
      
      How to use:
      - Use descriptive keys
      - Store as JSON for complex data
      - Include timestamps
      - Update rather than duplicate
      
      Keys to use:
      - "user_preferences": General preferences
      - "research_{topic}": Research findings
      - "context_{session_id}": Conversation context
```

### Tool Combination Strategies

Guide agents on using tools together.

```yaml
instructions:
  - |
      Multi-Tool Strategies:
      
      **Cache-Then-Fetch Pattern:**
      1. Check memory-server for cached data
      2. If cache hit and fresh → use cached data
      3. If cache miss or stale → fetch from source tool
      4. Update cache in memory-server
      5. Use fresh data
      
      Example:
      - Check memory-server for "weather_{location}_{date}"
      - If found and < 1 hour old, use it
      - Otherwise, call weather-tool
      - Store result in memory-server
      
      **Research-and-Store Pattern:**
      1. Search with duckduckgo
      2. Analyze results
      3. Store findings in memory-server
      4. Return synthesized response
      
      **Notify-and-Log Pattern:**
      1. Send notification via notification-server
      2. Store notification details in memory-server
      3. Track success/failure
      4. Don't duplicate notifications
```

### Error Handling Instructions

Guide agents on tool failures.

```yaml
instructions:
  - |
      Tool Error Handling:
      
      **If duckduckgo fails:**
      - Inform user: "Web search is temporarily unavailable"
      - Use training data knowledge where appropriate
      - Add caveat: "This information may not be current"
      - Don't block the response completely
      
      **If memory-server fails:**
      - Continue without stored preferences
      - Ask user for preferences directly
      - Log the error
      - Retry important operations once
      - Don't fail the entire request
      
      **If notification-server fails:**
      - Present information in chat instead
      - Retry once after brief delay
      - Log the failure
      - Inform user if notification couldn't be sent
      
      **If weather-tool fails:**
      - Inform user: "Weather data unavailable"
      - Offer to try again
      - Provide last known data if available
      - Don't speculate about weather
      
      **General Principle:**
      Degrade gracefully - never block the entire user request due to 
      a single tool failure. Provide the best possible experience with 
      available tools.
```

## Tool Performance Optimization

### Caching Strategy

Reduce redundant tool calls with caching.

```yaml
cached_researcher:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Smart Caching Strategy:
        
        **Before web search:**
        1. Check memory-server for "search_cache_{query_hash}"
        2. If found and < 24 hours old:
           - Use cached results
           - Note to user: "Based on recent search"
        3. If not found or stale:
           - Perform new search
           - Cache results with timestamp
           - Use fresh results
        
        **Cache Management:**
        - TTL: 24 hours for general searches
        - TTL: 1 hour for news/current events
        - TTL: 1 week for static reference information
        - Store: query, results, timestamp, source
        
        **Benefits:**
        - Faster responses
        - Reduced API costs
        - Consistent results within session
```

### Parallel Tool Calls

Use tools concurrently when possible.

```yaml
parallel_data_gatherer:
  toolsets:
    - weather-tool
    - google-calendar
    - memory-server
  instructions:
    - |
        Parallel Tool Usage:
        
        **When tools are independent:**
        Make calls concurrently to reduce latency.
        
        Example - Morning Briefing:
        Fetch simultaneously:
        - Today's calendar from google-calendar
        - Today's weather from weather-tool
        - User preferences from memory-server
        
        Then synthesize all data into briefing.
        
        **When tools depend on each other:**
        Call sequentially.
        
        Example - Personalized Weather:
        1. First: Get location from memory-server
        2. Then: Get weather-tool forecast for that location
```

### Lazy Loading

Only call tools when absolutely needed.

```yaml
lazy_assistant:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Lazy Tool Loading:
        
        **Don't preemptively call tools "just in case"**
        
        ❌ Bad:
        - Load all preferences at start
        - Search web for every topic mentioned
        - Store everything in memory
        
        ✅ Good:
        - Load preferences only when needed for decision
        - Search web only when training data insufficient
        - Store only important/reusable information
        
        **Example:**
        User: "What's 2 + 2?"
        - Don't search web (simple math)
        - Don't load preferences (not relevant)
        - Just respond: "4"
        
        User: "What are current mortgage rates?"
        - Do search web (current data needed)
        - Don't load preferences (first mention of finance)
        - Respond with findings
        
        User: "Find me a recipe for dinner"
        - Do load dietary preferences from memory
        - Do search web for recipes
        - Filter results by preferences
```

## Advanced Patterns

### Tool Chaining

Create sequences of tool operations.

```yaml
content_creator:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Content Creation Chain:
        
        **Step 1: Research**
        - Search duckduckgo for topic
        - Gather facts, statistics, quotes
        - Store in memory-server: "research_{topic}"
        
        **Step 2: Outline**
        - Load research from memory-server
        - Create content outline
        - Store in memory-server: "outline_{topic}"
        
        **Step 3: Draft**
        - Load outline from memory-server
        - Load research from memory-server
        - Write full draft
        - Store in memory-server: "draft_{topic}_v1"
        
        **Step 4: Revision**
        - Load draft from memory-server
        - Apply improvements
        - Store in memory-server: "draft_{topic}_v2"
        
        Each step builds on previous via memory-server.
```

### Conditional Tool Usage

Use tools based on settings or context.

```yaml
adaptive_assistant:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - if: "settings.research_mode"
      content: |
        RESEARCH MODE ACTIVE
        
        - Use duckduckgo liberally for fact-checking
        - Store all findings in memory-server
        - Provide extensive citations
        - Cross-reference multiple sources
    
    - "!settings.research_mode": |
        STANDARD MODE
        
        - Use duckduckgo only when necessary
        - Focus on concise responses
        - Store only important context in memory-server
    
    - |
        Adaptive tool usage based on mode setting.
```

### Tool Feedback Loops

Use tool results to guide further tool usage.

```yaml
iterative_researcher:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Iterative Research:
        
        **Initial Search:**
        1. Search duckduckgo for main topic
        2. Analyze results for:
           - Sufficient information?
           - Conflicting information?
           - Gaps in coverage?
           - Related subtopics?
        
        **Follow-up Searches:**
        Based on initial results:
        - If conflicts found: Search for authoritative sources
        - If gaps identified: Search specific subtopics
        - If outdated: Add date filters to search
        - If insufficient: Try alternative search terms
        
        **Convergence:**
        Continue until:
        - Sufficient information gathered
        - No significant conflicts
        - All key aspects covered
        - Or maximum 5 searches reached
        
        **Store in memory-server:**
        - All search queries tried
        - Key findings from each
        - Final synthesized research
```

### Tool Result Validation

Verify and validate tool results.

```yaml
validated_assistant:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Tool Result Validation:
        
        **For duckduckgo results:**
        Validate:
        - Are sources credible? (check domains)
        - Is information consistent across sources?
        - Are dates recent enough?
        - Are there conflicts?
        
        Quality tiers:
        - High: .edu, .gov, major news, peer-reviewed
        - Medium: Established publications, recognized experts
        - Low: Blogs, forums, unknown sources
        
        Prefer high-quality sources. Note when only low-quality available.
        
        **For memory-server results:**
        Validate:
        - Is data format correct?
        - Is timestamp reasonable (not too old)?
        - Is data complete (no missing fields)?
        
        If validation fails:
        - Fetch fresh data
        - Update cache
        - Log validation failure
```

## Tool Security Best Practices

### Data Privacy

Protect user data when using tools.

```yaml
privacy_conscious_agent:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Privacy Guidelines:
        
        **Web Search (duckduckgo):**
        ❌ Never include in search queries:
        - User names
        - Email addresses
        - Phone numbers
        - Credit card info
        - Personal identifiers
        
        ✅ Generalize queries:
        - "Best Italian restaurants" not "Best near John Smith"
        - "Child nutrition" not "nutrition for 8-year-old Jenny"
        
        **Data Storage (memory-server):**
        ❌ Never store:
        - Passwords or credentials
        - Credit card numbers
        - Social security numbers
        - Health records
        - Other sensitive PII
        
        ✅ Safe to store:
        - User preferences
        - Settings and configuration
        - Non-sensitive context
        - Pseudonymized data
```

### Rate Limiting

Respect tool rate limits.

```yaml
rate_limited_agent:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Rate Limiting:
        
        **Track usage:**
        Store in memory-server:
        - "tool_usage_duckduckgo": Array of timestamps
        - "tool_usage_weather": Array of timestamps
        
        **Before tool call:**
        1. Load usage history from memory-server
        2. Count calls in last minute
        3. If limit reached:
           - Wait or return error
           - Inform user of limit
           - Suggest alternative
        4. If under limit:
           - Make call
           - Add timestamp to history
           - Store updated history
        
        **Rate Limits:**
        - duckduckgo: 10 calls/minute
        - weather-tool: 60 calls/minute
        - memory-server: 100 calls/minute
        - notification-server: 10 calls/minute
```

### Error Recovery

Handle tool errors gracefully.

```yaml
resilient_agent:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Error Recovery Strategy:
        
        **Retry Logic:**
        - Network errors: Retry once after 1 second
        - Rate limit errors: Wait and retry
        - Auth errors: Don't retry, report issue
        - Other errors: Retry once
        
        **Fallback Strategy:**
        Level 1: Try primary tool
        Level 2: Try cached data (if available)
        Level 3: Use training data knowledge
        Level 4: Inform user of limitation
        
        **Never:**
        - Fail silently
        - Block entire response
        - Retry endlessly
        - Expose internal error details to user
        
        **Always:**
        - Log errors for debugging
        - Inform user gracefully
        - Provide best alternative
        - Continue with available capabilities
```

## Testing Tools

### Tool Integration Testing

Test tools work as expected.

```yaml
tool_test_agent:
  toolsets:
    - duckduckgo
    - memory-server
  instructions:
    - |
        Tool Testing Checklist:
        
        **Basic Functionality:**
        - Can call tool successfully?
        - Does tool return expected format?
        - Are results reasonable?
        
        **Error Handling:**
        - What happens if tool times out?
        - What happens if tool returns error?
        - What happens if tool unavailable?
        
        **Performance:**
        - What is average response time?
        - Does tool handle concurrent calls?
        - Are there rate limits?
        
        **Integration:**
        - Can combine with other tools?
        - Can chain tool calls?
        - Can cache tool results?
```

## Common Pitfalls

### Over-reliance on Tools

❌ **Don't call tools unnecessarily:**

```yaml
# Bad - searches web for everything
bad_agent:
  instructions:
    - |
        For every question, search the web for an answer.
```

✅ **Use tools when needed:**

```yaml
# Good - uses web search appropriately
good_agent:
  instructions:
    - |
        Use web search when:
        - Question requires current information
        - Training data is insufficient
        - Facts need verification
        
        Otherwise, use training knowledge.
```

### Ignoring Tool Failures

❌ **Don't assume tools always work:**

```yaml
# Bad - no error handling
bad_agent:
  instructions:
    - |
        Search web for answer. Present results.
```

✅ **Handle errors gracefully:**

```yaml
# Good - handles failures
good_agent:
  instructions:
    - |
        Try to search web. If search fails:
        - Inform user search is unavailable
        - Provide answer from training data
        - Note answer may not be current
```

### Poor Key Management

❌ **Don't use generic keys:**

```yaml
# Bad - unclear keys
bad_keys:
  - "data"
  - "info"
  - "stuff"
```

✅ **Use descriptive keys:**

```yaml
# Good - clear, structured keys
good_keys:
  - "user_preferences"
  - "research_ai_ethics_2024"
  - "session_context_abc123"
```

## Next Steps

- **[Available Tools](available-tools.md)** - Complete tool catalog
- **[Tools Overview](overview.md)** - Understanding tools
- **[Agent Examples](../agents/examples.md)** - Teams using tools

---

**Related:**  
[Overview](overview.md) | [Available Tools](available-tools.md) | [Creating Teams](../agents/creating-agents.md)
