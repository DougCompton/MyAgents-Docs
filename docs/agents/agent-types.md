# Agent Execution Types

MyAgents supports four powerful agent execution types for different workflow patterns.

## Overview

| Type | Purpose | Use Case |
|------|---------|----------|
| [LLM](#llm-agent) | Interactive AI assistant | Chat, analysis, content creation |
| [Parallel](#parallel-agent) | Concurrent execution | Multi-perspective analysis, parallel research |
| [Loop](#loop-agent) | Iterative refinement | Content improvement, retry logic |
| [Sequence](#sequence-agent) | Sequential pipeline | ETL workflows, multi-stage processing |

## LLM Agent

The standard interactive AI agent with LLM capabilities.

### Structure

```json
{
  "type": "llm",
  "name": "Assistant",
  "description": "General purpose assistant",
  "instructions": [
    {"content": "You are a helpful assistant..."}
  ],
  "model": "anthropic",
  "toolsets": ["duckduckgo"]
}
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | No | "llm" (default if omitted) |
| `name` | string | Yes | Agent identifier |
| `description` | string | Yes | Agent purpose |
| `instructions` | array | Yes | Behavior instructions |
| `model` | string | No | LLM provider |
| `toolsets` | array | No | Available tools |
| `subAgents` | array | No | Delegatable agents |

### Features

- ✅ **Interactive conversations** - Maintains context across messages
- ✅ **Tool integration** - Access to MCP tools
- ✅ **Streaming support** - Real-time response streaming
- ✅ **Conditional instructions** - Dynamic behavior based on settings
- ✅ **Sub-agent delegation** - Can delegate via `transfer_task`

### When to Use

- Chat interfaces
- Content generation
- Data analysis
- Q&A systems
- Task automation

---

## Parallel Agent

Executes multiple sub-agents concurrently for parallel processing.

### Structure

```json
{
  "type": "parallel",
  "name": "Research Team",
  "description": "Multi-perspective analysis",
  "sub_agents": ["analyst1", "analyst2", "analyst3"],
  "max_concurrency": 2
}
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | Yes | "parallel" |
| `name` | string | Yes | Agent identifier |
| `description` | string | Yes | Agent purpose |
| `sub_agents` | array | Yes | List of agents to run in parallel |
| `max_concurrency` | number | No | Max parallel executions (default: all) |

### Behavior

**Input:** Single string sent to all sub-agents simultaneously

**Processing:**
- All sub-agents receive the same input
- Execute concurrently (respecting `max_concurrency`)
- Continue on partial failures

**Output:** JSON array of results:
```json
[
  "Result from analyst1",
  "Result from analyst2",
  "Result from analyst3"
]
```

### Example: Multi-Perspective Research

```json
{
  "version": "1",
  "id": "parallel-research",
  "name": "Parallel Research Team",
  "description": "Analyzes topics from multiple perspectives simultaneously",
  "default_agent": "coordinator",
  "agents": {
    "coordinator": {
      "type": "parallel",
      "name": "Research Coordinator",
      "description": "Coordinates parallel research",
      "sub_agents": [
        "technical_researcher",
        "business_researcher",
        "academic_researcher"
      ],
      "max_concurrency": 3
    },
    "technical_researcher": {
      "type": "llm",
      "name": "Technical Researcher",
      "description": "Technical analysis",
      "instructions": [
        {
          "": "Analyze from technical perspective: architecture, performance, security"
        }
      ]
    },
    "business_researcher": {
      "type": "llm",
      "name": "Business Researcher",
      "description": "Business analysis",
      "instructions": [
        {
          "": "Analyze from business perspective: market, ROI, strategy"
        }
      ]
    },
    "academic_researcher": {
      "type": "llm",
      "name": "Academic Researcher",
      "description": "Academic analysis",
      "instructions": [
        {
          "": "Analyze from academic perspective: theory, research, citations"
        }
      ]
    }
  }
}
```

**User Input:** "Analyze the impact of AI on healthcare"

**Execution:**
```
Input sent to all 3 researchers simultaneously:
├─→ Technical Researcher (concurrent)
├─→ Business Researcher (concurrent)
└─→ Academic Researcher (concurrent)

Results combined:
[
  "Technical: AI enables...",
  "Business: Market opportunity...",
  "Academic: Research shows..."
]
```

### When to Use

- **Multi-perspective analysis** - Get different viewpoints on same input
- **Parallel research** - Multiple specialists work simultaneously
- **A/B testing** - Compare different approaches
- **Consensus building** - Gather multiple opinions
- **Distributed processing** - Split workload across agents

### Performance Considerations

**Concurrency Control:**
```json
{
  "max_concurrency": 2  // Limit to 2 simultaneous executions
}
```

Without limit, all sub-agents run simultaneously (fastest but more resource-intensive).

**Error Handling:**
- Partial failures logged but don't stop other agents
- Successful results returned even if some agents fail
- Check logs for detailed error information

---

## Loop Agent

Iteratively executes a sub-agent for refinement or retry logic.

### Structure

```json
{
  "type": "loop",
  "name": "Refiner",
  "description": "Iteratively improves content",
  "sub_agent": "editor",
  "loop": {
    "mode": "fixed",
    "max_iterations": 3,
    "exit_on_error": true
  }
}
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | Yes | "loop" |
| `name` | string | Yes | Agent identifier |
| `description` | string | Yes | Agent purpose |
| `sub_agent` | string | Yes | Agent to execute repeatedly |
| `loop` | object | Yes | Loop configuration |

### Loop Configuration

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `mode` | string | Yes | "fixed" or "until_success" |
| `max_iterations` | number | Yes | Maximum iterations (1-100) |
| `success_criteria` | array | No | Success check conditions |
| `exit_on_error` | boolean | No | Stop on error (default: true) |

### Modes

#### Fixed Mode

Run exact number of iterations regardless of results.

```json
{
  "mode": "fixed",
  "max_iterations": 3
}
```

**Execution:**
```
Iteration 1: Input → Agent → Output 1
Iteration 2: Output 1 → Agent → Output 2
Iteration 3: Output 2 → Agent → Final Output
```

Each iteration's output becomes next iteration's input.

#### Until Success Mode

Run until success criteria met or max iterations reached.

```json
{
  "mode": "until_success",
  "max_iterations": 5,
  "success_criteria": ["approved", "complete", "valid"]
}
```

**Execution:**
- Checks if output contains any success criteria keywords
- Stops early if criteria met
- Otherwise continues to max iterations

### Example: Iterative Content Refinement

```json
{
  "version": "1",
  "id": "loop-refine",
  "name": "Iterative Refinement Agent",
  "description": "Iteratively refines content through multiple passes",
  "default_agent": "refiner",
  "agents": {
    "refiner": {
      "type": "loop",
      "name": "Content Refiner",
      "description": "Iteratively refines content",
      "sub_agent": "editor",
      "loop": {
        "mode": "fixed",
        "max_iterations": 3,
        "exit_on_error": true
      }
    },
    "editor": {
      "type": "llm",
      "name": "Content Editor",
      "description": "Edits and improves content",
      "instructions": [
        {
          "": "Improve the text by:\n1. Fix grammar\n2. Improve clarity\n3. Enhance structure\n4. Make more concise\n5. Strengthen arguments\n\nProvide improved version."
        }
      ]
    }
  }
}
```

**User Input:** "This text needs improvement. It has errors and is unclear..."

**Execution:**
```
Pass 1: Original text → Editor → Improved v1
Pass 2: Improved v1 → Editor → Improved v2
Pass 3: Improved v2 → Editor → Final polished text
```

Each pass progressively refines the content.

### When to Use

- **Content refinement** - Progressive improvement through iterations
- **Retry logic** - Retry operations until success
- **Quality improvement** - Iteratively enhance outputs
- **Progressive processing** - Build up complexity through passes
- **Validation loops** - Retry until criteria met

### Best Practices

**Set reasonable iteration limits:**
```json
{
  "max_iterations": 3  // ✓ Good for refinement
}
```

```json
{
  "max_iterations": 100  // ✗ Too many, may be inefficient
}
```

**Use exit_on_error for robustness:**
```json
{
  "exit_on_error": true  // Stop if agent fails
}
```

---

## Sequence Agent

Executes sub-agents sequentially in a pipeline, passing output to next input.

### Structure

```json
{
  "type": "sequence",
  "name": "Pipeline",
  "description": "Sequential processing pipeline",
  "sub_agents": ["stage1", "stage2", "stage3"]
}
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | Yes | "sequence" |
| `name` | string | Yes | Agent identifier |
| `description` | string | Yes | Agent purpose |
| `sub_agents` | array | Yes | Agents to run in order |

### Behavior

**Input:** Initial string sent to first agent

**Processing:**
- Agent 1 processes input → Output 1
- Agent 2 receives Output 1 → Output 2
- Agent 3 receives Output 2 → Final Output

**Output:** Output from final agent in sequence

**Error Handling:** Fails fast on first error with detailed context

### Example: ETL Pipeline

```json
{
  "version": "1",
  "id": "sequence-pipeline",
  "name": "Content Processing Pipeline",
  "description": "Sequential pipeline for extraction, analysis, formatting",
  "default_agent": "pipeline",
  "agents": {
    "pipeline": {
      "type": "sequence",
      "name": "Content Pipeline",
      "description": "Processes content through stages",
      "sub_agents": ["extractor", "analyzer", "formatter"]
    },
    "extractor": {
      "type": "llm",
      "name": "Information Extractor",
      "description": "Extracts key information",
      "instructions": [
        {
          "": "Extract key information:\n- Main topics\n- Facts and data\n- Arguments\n- Evidence\n- Conclusions\n\nOutput structured format."
        }
      ]
    },
    "analyzer": {
      "type": "llm",
      "name": "Content Analyzer",
      "description": "Analyzes extracted information",
      "instructions": [
        {
          "": "Analyze the extracted information:\n- Identify patterns\n- Assess strengths/weaknesses\n- Note gaps\n- Provide insights\n- Suggest implications"
        }
      ]
    },
    "formatter": {
      "type": "llm",
      "name": "Report Formatter",
      "description": "Formats into professional report",
      "instructions": [
        {
          "": "Format into professional report:\n- Executive summary\n- Clear headings\n- Markdown formatting\n- Lists\n- Conclusion"
        }
      ]
    }
  }
}
```

**User Input:** "Here is raw customer feedback data..."

**Execution:**
```
Step 1 (Extract): Raw data → Extractor → Structured bullet points
Step 2 (Analyze): Bullet points → Analyzer → Insights and patterns
Step 3 (Format): Insights → Formatter → Professional report
```

Each stage transforms the output for the next stage.

### When to Use

- **ETL workflows** - Extract, Transform, Load pipelines
- **Multi-stage processing** - Progressive transformations
- **Content pipelines** - Research → Write → Edit
- **Data transformations** - Parse → Analyze → Format
- **Quality gates** - Validate → Process → Verify

### Common Patterns

**Research Pipeline:**
```json
{
  "sub_agents": ["researcher", "writer", "editor"]
}
```

**Data Processing:**
```json
{
  "sub_agents": ["validator", "transformer", "summarizer"]
}
```

**Quality Pipeline:**
```json
{
  "sub_agents": ["generator", "reviewer", "approver"]
}
```

---

## Composability

Agent types can be nested and combined for powerful workflows.

### Parallel + LLM

```json
{
  "coordinator": {
    "type": "parallel",
    "sub_agents": ["llm1", "llm2", "llm3"]
  }
}
```

### Sequence + Parallel

```json
{
  "pipeline": {
    "type": "sequence",
    "sub_agents": ["collector", "parallel_analyzer", "synthesizer"]
  },
  "parallel_analyzer": {
    "type": "parallel",
    "sub_agents": ["analyst1", "analyst2"]
  }
}
```

### Loop + Sequence

```json
{
  "refiner": {
    "type": "loop",
    "sub_agent": "improvement_pipeline",
    "loop": {"mode": "fixed", "max_iterations": 3}
  },
  "improvement_pipeline": {
    "type": "sequence",
    "sub_agents": ["grammar_check", "style_enhance"]
  }
}
```

## Circular Reference Protection

The system prevents infinite loops:

```json
❌ // This would fail
{
  "agent_a": {
    "type": "sequence",
    "sub_agents": ["agent_b"]
  },
  "agent_b": {
    "type": "sequence",
    "sub_agents": ["agent_a"]  // Circular!
  }
}
```

**Error:** `Circular reference detected: agent_a -> agent_b -> agent_a`

---

## Performance Tips

### Parallel Agents

- Use `max_concurrency` to limit resource usage
- Balance speed vs. cost (more concurrent = more API calls)
- Monitor total execution time

### Loop Agents

- Keep `max_iterations` reasonable (typically 3-5)
- Use `exit_on_error: true` for reliability
- Consider cost of multiple iterations

### Sequence Agents

- Minimize number of stages (each adds latency)
- Consider combining simple stages
- Ensure clear output formatting between stages

---

## Next Steps

- **[Creating Agents](creating-agents.md)** - Build your own agents
- **[Examples](examples.md)** - More real-world templates
- **[Multi-Persona Workflows](../advanced/multi-persona.md)** - Advanced patterns

---

**Related:**  
[Agent Overview](overview.md) | [Creating Agents](creating-agents.md) | [Configuration](configuration.md)

