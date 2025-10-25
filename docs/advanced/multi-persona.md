# Multi-Persona Workflows

Master complex agent orchestration with multiple specialized personas.

## Overview

**Multi-persona agents** combine specialized roles to handle complex workflows that require:

- Domain expertise across multiple areas
- Sequential processing steps
- Parallel specialist consultation
- Quality control and review
- Coordinated task execution

## Architecture Patterns

### Pattern 1: Coordinator-Specialist

One coordinator manages multiple specialists.

```
┌──────────────────┐
│   Coordinator    │
└────────┬─────────┘
         │
    ┌────┴────┬────────┬────────┐
    │         │        │        │
┌───▼───┐ ┌──▼───┐ ┌──▼───┐ ┌──▼───┐
│Spec A │ │Spec B│ │Spec C│ │Spec D│
└───────┘ └──────┘ └──────┘ └──────┘
```

**Example:**

```json
{
  "id": "support-router",
  "name": "Customer Support Router",
  "description": "Routes support tickets to appropriate specialists",
  "interactive": true,
  "agents": [
    {
      "name": "coordinator",
      "description": "Routes tickets to specialists",
      "instructions": [
        {
          "content": "Analyze support requests and delegate to:\n- technical_support for bugs and technical issues\n- billing_support for payment questions\n- account_support for account access\n\nUse transfer_task to delegate."
        }
      ],
      "subAgents": ["technical_support", "billing_support", "account_support"]
    },
    {
      "name": "technical_support",
      "description": "Handles technical issues",
      "instructions": [
        {"content": "Provide technical troubleshooting..."}
      ]
    },
    {
      "name": "billing_support",
      "description": "Handles billing inquiries",
      "instructions": [
        {"content": "Address payment questions..."}
      ]
    },
    {
      "name": "account_support",
      "description": "Handles account management",
      "instructions": [
        {"content": "Assist with account settings..."}
      ]
    }
  ]
}
```

**When to use:**
- Support routing
- Request triage
- Domain-specific consultation
- Parallel specialist input

---

### Pattern 2: Sequential Pipeline

Tasks flow through defined sequence.

```
Request → Step 1 → Step 2 → Step 3 → Result
```

**Example: Content Creation**

```json
{
  "id": "content-pipeline",
  "name": "Content Creation Pipeline",
  "description": "Creates content through research, writing, and editing stages",
  "interactive": true,
  "agents": [
    {
      "name": "project_manager",
      "description": "Manages content creation workflow",
      "instructions": [
        {
          "content": "Coordinate content creation:\n1. Delegate to researcher for information gathering\n2. Send research to writer for content creation\n3. Send draft to editor for review\n4. Present final content\n\nEnsure quality at each stage."
        }
      ],
      "subAgents": ["researcher", "writer", "editor"]
    },
    {
      "name": "researcher",
      "description": "Gathers information",
      "instructions": [
        {"content": "Research topic thoroughly. Find facts, statistics, expert opinions. Cite all sources."}
      ],
      "toolsets": ["duckduckgo"]
    },
    {
      "name": "writer",
      "description": "Creates content",
      "instructions": [
        {"content": "Write engaging content based on research. Use clear structure: intro, body, conclusion."}
      ]
    },
    {
      "name": "editor",
      "description": "Reviews and improves",
      "instructions": [
        {"content": "Edit for grammar, clarity, flow, and accuracy. Note improvements made."}
      ]
    }
  ]
}
```

**When to use:**
- Content creation (research → write → edit)
- Order processing (validate → process → fulfill)
- Report generation (collect → analyze → format)
- Quality workflows (create → review → approve)

---

### Pattern 3: Hierarchical Delegation

Multi-level coordination.

```
             Project Manager
                    │
        ┌───────────┼───────────┐
        │                       │
   Team Lead A            Team Lead B
        │                       │
   ┌────┴────┐             ┌────┴────┐
   │         │             │         │
Worker A1  Worker A2   Worker B1  Worker B2
```

**Example: Research Project**

```json
{
  "id": "research-project",
  "name": "Research Project Manager",
  "description": "Manages complex research projects with multiple teams",
  "interactive": true,
  "agents": [
    {
      "name": "project_manager",
      "description": "Oversees entire project",
      "instructions": [
        {"content": "Coordinate research project:\n1. Define research scope\n2. Delegate to team leads\n3. Synthesize findings\n4. Present comprehensive report"}
      ],
      "subAgents": ["data_team_lead", "analysis_team_lead"]
    },
    {
      "name": "data_team_lead",
      "description": "Manages data collection team",
      "instructions": [
        {"content": "Coordinate data collection:\n1. Delegate to web_researcher\n2. Delegate to data_analyst\n3. Compile findings"}
      ],
      "subAgents": ["web_researcher", "data_analyst"]
    },
    {
      "name": "analysis_team_lead",
      "description": "Manages analysis team",
      "instructions": [
        {"content": "Coordinate analysis:\n1. Receive data from data team\n2. Delegate analysis tasks\n3. Generate insights"}
      ],
      "subAgents": ["statistical_analyst", "report_writer"]
    },
    {
      "name": "web_researcher",
      "description": "Researches online sources",
      "instructions": [{"content": "Search and gather information..."}],
      "toolsets": ["duckduckgo"]
    },
    {
      "name": "data_analyst",
      "description": "Analyzes collected data",
      "instructions": [{"content": "Process and analyze data..."}]
    },
    {
      "name": "statistical_analyst",
      "description": "Performs statistical analysis",
      "instructions": [{"content": "Apply statistical methods..."}]
    },
    {
      "name": "report_writer",
      "description": "Writes final report",
      "instructions": [{"content": "Create comprehensive report..."}]
    }
  ]
}
```

**When to use:**
- Large projects
- Complex organizations
- Multi-stage workflows
- Enterprise systems

---

## Delegation Mechanics

### Using transfer_task

The `transfer_task` tool is automatically available to personas with `subAgents`.

**Coordinator instructions:**

```json
{
  "name": "coordinator",
  "subAgents": ["specialist_a", "specialist_b"],
  "instructions": [
    {
      "content": "Coordinate tasks:\n\n**Delegation:**\n1. Analyze request to determine which specialist is needed\n2. Use transfer_task with:\n   - target_persona: Name of specialist\n   - task_description: Clear description of what to do\n3. Wait for specialist to complete\n4. Process results and present to user\n\n**Available specialists:**\n- specialist_a: Handles X type tasks\n- specialist_b: Handles Y type tasks"
    }
  ]
}
```

### Delegation Best Practices

**DO:**
- Provide clear task descriptions
- One specialist at a time
- Wait for completion before next delegation
- Process specialist results before presenting to user

**DON'T:**
- Delegate to unavailable personas
- Try to delegate multiple tasks simultaneously
- Skip result processing
- Delegate without clear purpose

---

## Tool Distribution

### Who Gets Tools?

**Coordinators:** Usually no tools
- Just delegate via `transfer_task`
- Focus on workflow management

**Specialists:** Tools as needed
- Each specialist gets required tools
- Minimal tool overlap

**Example:**

```json
{
  "agents": [
    {
      "name": "coordinator",
      "subAgents": ["researcher", "writer"],
      "toolsets": []  // No tools needed
    },
    {
      "name": "researcher",
      "toolsets": ["duckduckgo"]  // Needs web search
    },
    {
      "name": "writer",
      "toolsets": []  // No tools needed, uses research
    }
  ]
}
```

---

## State Management

### Context Preservation

Specialists inherit context from coordinator:
- User's original request
- Conversation history
- Previous specialist outputs

### Passing Information

**Coordinator receives specialist output:**

```
User: "Write a blog post about AI"
→ Coordinator delegates to researcher
← Researcher returns findings
→ Coordinator delegates to writer with findings
← Writer returns draft
→ Coordinator presents to user
```

**Specialists don't directly interact:**

```
❌ Researcher → Writer (direct)
✅ Researcher → Coordinator → Writer (via coordinator)
```

---

## Error Handling

### Specialist Failures

Coordinators should handle specialist errors:

```json
{
  "name": "coordinator",
  "instructions": [
    {
      "content": "Error handling:\n\n**If specialist fails:**\n1. Acknowledge the issue to user\n2. Try alternative approach if available\n3. Provide partial results if possible\n4. Escalate if necessary\n\n**Don't:**\n- Expose internal errors to user\n- Retry indefinitely\n- Proceed without required information"
    }
  ]
}
```

### Validation

Coordinator validates specialist output:

```json
{
  "content": "After receiving specialist results:\n\n1. **Validate:**\n   - Is output complete?\n   - Does it address the task?\n   - Is quality acceptable?\n\n2. **If invalid:**\n   - Request clarification/revision\n   - Provide specific feedback\n   - Re-delegate if needed\n\n3. **If valid:**\n   - Process results\n   - Continue workflow\n   - Present to user"
}
```

---

## Performance Optimization

### Minimize Delegations

**Inefficient:**
```
Coordinator → A → Coordinator → B → Coordinator → C
(3 delegations)
```

**Efficient:**
```
Coordinator → A (includes B and C work)
(1 delegation)
```

### Batch Related Tasks

Combine similar tasks into one delegation:

**Before:**
```
1. Delegate: "Search for product reviews"
2. Delegate: "Search for product pricing"
3. Delegate: "Search for product availability"
```

**After:**
```
1. Delegate: "Search for product reviews, pricing, and availability"
```

---

## Testing Multi-Persona Agents

### Test Each Persona

1. Test base instructions of each persona
2. Verify tools work for each specialist
3. Check error handling at each level

### Test Delegation Paths

1. Test each delegation route
2. Verify context preservation
3. Check result passing
4. Test error scenarios

### Test Complete Workflows

1. End-to-end testing
2. Multiple user requests in sequence
3. Edge cases and error conditions
4. Performance under load

---

## Common Patterns Library

### Research-Write-Edit

```json
{
  "agents": [
    {"name": "coordinator", "subAgents": ["researcher", "writer", "editor"]},
    {"name": "researcher", "toolsets": ["duckduckgo"]},
    {"name": "writer"},
    {"name": "editor"}
  ]
}
```

### Validate-Process-Notify

```json
{
  "agents": [
    {"name": "coordinator", "subAgents": ["validator", "processor", "notifier"]},
    {"name": "validator"},
    {"name": "processor", "toolsets": ["memory-server"]},
    {"name": "notifier", "toolsets": ["notification-server"]}
  ]
}
```

### Triage-Route-Resolve

```json
{
  "agents": [
    {"name": "triage", "subAgents": ["tech_support", "billing", "account"]},
    {"name": "tech_support"},
    {"name": "billing"},
    {"name": "account"}
  ]
}
```

---

## Next Steps

- **[Agent Examples](../agents/examples.md)** - Complete multi-persona examples
- **[Creating Agents](../agents/creating-agents.md)** - Build your own
- **[Settings](settings.md)** - Configure behavior dynamically

---

**Related:**  
[Agent Overview](../agents/overview.md) | [Agent Examples](../agents/examples.md) | [Creating Agents](../agents/creating-agents.md)

