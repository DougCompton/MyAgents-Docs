# Agent Types

MyAgents supports four execution types that enable different processing patterns for your AI teams.

## Overview

| Type | Purpose | Use Cases |
|------|---------|-----------|
| **LLM Agent** | Interactive AI assistant | Conversations, analysis, content creation |
| **Parallel Agent** | Concurrent execution | Multi-perspective analysis, batch processing |
| **Loop Agent** | Iterative refinement | Content improvement, problem solving |
| **Sequence Agent** | Sequential pipeline | Multi-stage workflows, progressive processing |

## LLM Agent

The **LLM Agent** is the default agent type - a standard interactive AI assistant powered by large language models.

### Characteristics

- ✅ Interactive conversations with context
- ✅ Access to tools and external systems
- ✅ Can delegate to other agents
- ✅ Full instruction customization
- ✅ Streaming responses

### Configuration

```yaml
agents:
  assistant:
    type: llm  # Can be omitted (default)
    name: Research Assistant
    description: Conducts web research and analysis
    model: anthropic
    instructions:
      - |
        You are a research assistant. Conduct thorough web research 
          using available tools. Provide well-sourced, accurate information 
          with citations.
    toolsets:
      - duckduckgo
      - memory-server
```

### Key Properties

| Property | Required | Description |
|----------|----------|-------------|
| `type` | No | Set to `"llm"` or omit (default) |
| `name` | Yes | Agent display name |
| `description` | Yes | Brief role description |
| `model` | No | LLM provider: `"anthropic"` or `"gpt-4o"` |
| `instructions` | Yes | Behavioral guidelines |
| `toolsets` | No | Available tool access |
| `sub_agents` | No | Agents this can delegate to |

### Use Cases

**Content Creation**
```yaml
content_writer:
  type: llm
  name: Content Writer
  description: Creates blog posts and articles
  model: anthropic
  instructions:
    - |
      Create engaging, SEO-optimized content. Use clear structure 
        with headings, bullet points, and examples. Target 1000-1500 words.
  toolsets:
    - duckduckgo
```

**Customer Support**
```yaml
support_agent:
  type: llm
  name: Support Agent
  description: Handles customer inquiries
  instructions:
    - |
      Provide friendly, helpful support. Check knowledge base first,  then escalate complex issues to technical team.
  sub_agents:
    - technical_specialist
  toolsets:
    - memory-server
```

**Data Analysis**
```yaml
analyst:
  type: llm
  name: Data Analyst
  description: Analyzes datasets and generates insights
  model: gpt-4o
  instructions:
    - |
      Analyze provided data thoroughly. Identify trends, anomalies, 
        and actionable insights. Present findings with visualizations.
```

### Best Practices

1. **Clear Instructions**: Define scope, constraints, and output format
2. **Appropriate Tools**: Only include tools the agent needs
3. **Model Selection**: Use Claude for analysis/writing, GPT-4 for general tasks
4. **Sub-Agent Design**: Structure delegations hierarchically

## Parallel Agent

The **Parallel Agent** executes multiple sub-agents concurrently and combines their results.

### Characteristics

- ✅ Concurrent execution of sub-agents
- ✅ Results combined automatically
- ✅ Ideal for multi-perspective analysis
- ✅ Reduces total processing time
- ⚠️ Does not support tools directly
- ⚠️ Cannot be interactive

### Configuration

```yaml
agents:
  multi_perspective_analyzer:
    type: parallel
    name: Multi-Perspective Analyzer
    description: Analyzes content from multiple viewpoints simultaneously
    instructions:
      - |
          Combine insights from all perspectives into a comprehensive analysis. 
          Identify agreements, contradictions, and unique insights from each viewpoint.
    sub_agents:
      - technical_perspective
      - business_perspective
      - user_perspective
  
  technical_perspective:
    type: llm
    name: Technical Perspective
    description: Technical feasibility analysis
    instructions:
      - |
          Analyze from a technical standpoint: implementation complexity, 
          scalability, maintainability, and technical risks.
  
  business_perspective:
    type: llm
    name: Business Perspective
    description: Business value analysis
    instructions:
      - |
          Analyze from a business standpoint: ROI, market opportunity, 
          competitive advantage, and resource requirements.
  
  user_perspective:
    type: llm
    name: User Perspective
    description: User experience analysis
    instructions:
      - |
          Analyze from a user standpoint: usability, accessibility, 
          user value, and adoption barriers.
```

### Key Properties

| Property | Required | Description |
|----------|----------|-------------|
| `type` | Yes | Must be `"parallel"` |
| `name` | Yes | Agent display name |
| `description` | Yes | Brief role description |
| `instructions` | Yes | How to combine sub-agent results |
| `sub_agents` | Yes | Array of agents to execute concurrently |
| `toolsets` | No | ⚠️ Not supported for parallel agents |

### Execution Flow

```
User Request
     │
     ▼
Parallel Agent
     │
     ├─────────┬─────────┬─────────┐
     ▼         ▼         ▼         ▼
Sub-Agent 1  Sub-Agent 2  Sub-Agent 3  Sub-Agent 4
     │         │         │         │
     │    (Execute Concurrently)  │
     │         │         │         │
     └─────────┴─────────┴─────────┘
              │
              ▼
      Combine Results
              │
              ▼
         Final Response
```

### Use Cases

**Product Review Analysis**
```yaml
product_reviewer:
  type: parallel
  name: Product Review Team
  description: Multi-dimensional product analysis
  instructions:
    - |
        Synthesize all perspectives into a balanced review with:
        - Overall recommendation
        - Key strengths and weaknesses
        - Best use cases
        - Alternative considerations
  sub_agents:
    - features_analyst
    - price_analyst
    - quality_analyst
    - support_analyst
```

**Content Fact-Checking**
```yaml
fact_checker:
  type: parallel
  name: Fact Checking Team
  description: Verifies claims from multiple sources
  instructions:
    - |
        Cross-reference all findings. Report verification status for each claim:
        - Confirmed (multiple sources agree)
        - Disputed (sources conflict)
        - Unverified (insufficient information)
  sub_agents:
    - source_checker_1
    - source_checker_2
    - source_checker_3
```

**Competitive Analysis**
```yaml
competitor_analyzer:
  type: parallel
  name: Competitive Analysis Team
  description: Analyzes multiple competitors simultaneously
  instructions:
    - |
        Create a competitive landscape summary:
        - Feature comparison matrix
        - Pricing positioning
        - Market differentiation
        - Strategic recommendations
  sub_agents:
    - competitor_a_analyst
    - competitor_b_analyst
    - competitor_c_analyst
```

### Best Practices

1. **Independent Sub-Agents**: Each sub-agent should work independently
2. **Clear Synthesis**: Main agent combines results coherently
3. **Balanced Load**: Sub-agents should have similar complexity
4. **Error Handling**: One sub-agent failure doesn't block others

### Limitations

- ❌ Cannot use `toolsets` directly (sub-agents can have tools)
- ❌ Cannot be set as `interactive: true`
- ❌ Sub-agents cannot communicate with each other during execution

## Loop Agent

The **Loop Agent** executes an agent iteratively to refine or improve content.

### Characteristics

- ✅ Iterative refinement and improvement
- ✅ Each iteration builds on previous results
- ✅ Configurable iteration count
- ✅ Automatic convergence
- ⚠️ Requires exactly one sub-agent
- ⚠️ Cannot use tools directly

### Configuration

```yaml
agents:
  content_refiner:
    type: loop
    name: Content Refiner
    description: Iteratively improves content quality
    max_iterations: 3
    instructions:
      - |
          Review the content from the previous iteration and:
          1. Identify areas for improvement
          2. Enhance clarity and readability
          3. Strengthen arguments
          4. Polish language and style
          
          On the final iteration, provide the polished final version.
    sub_agents:
      - content_editor
  
  content_editor:
    type: llm
    name: Content Editor
    description: Edits and refines content
    instructions:
      - |
          Edit the provided content for:
          - Grammar and spelling
          - Clarity and conciseness
          - Flow and structure
          - Impact and engagement
```

### Key Properties

| Property | Required | Description |
|----------|----------|-------------|
| `type` | Yes | Must be `"loop"` |
| `name` | Yes | Agent display name |
| `description` | Yes | Brief role description |
| `instructions` | Yes | What to do each iteration |
| `max_iterations` | Yes | Number of iterations (1-10) |
| `sub_agents` | Yes | Array with exactly one agent |
| `toolsets` | No | ⚠️ Not supported for loop agents |

### Execution Flow

```
User Request + Initial Content
         │
         ▼
    Iteration 1
         │
    [Sub-Agent]
         │
         ▼
    Iteration 2
         │
    [Sub-Agent] (receives previous output)
         │
         ▼
    Iteration 3
         │
    [Sub-Agent] (receives previous output)
         │
         ▼
   Final Result
```

### Use Cases

**Content Polish Loop**
```yaml
polish_loop:
  type: loop
  name: Content Polish Loop
  description: Progressively polishes content to perfection
  max_iterations: 3
  instructions:
    - |
        Iteration by iteration, improve:
        1. First pass: Structure and organization
        2. Second pass: Language and clarity
        3. Final pass: Style and impact
  sub_agents:
    - polisher

polisher:
  type: llm
  name: Content Polisher
  model: anthropic
  instructions:
    - |
        Improve the content while maintaining the core message.
        Focus on readability, engagement, and professionalism.
```

**Code Review Loop**
```yaml
code_reviewer:
  type: loop
  name: Code Review Loop
  description: Iteratively reviews and improves code
  max_iterations: 2
  instructions:
    - |
        Each iteration:
        1. Identify code smells and issues
        2. Suggest specific improvements
        3. Check against best practices
  sub_agents:
    - code_analyst

code_analyst:
  type: llm
  name: Code Analyst
  instructions:
    - |
        Analyze code for:
        - Correctness and bugs
        - Performance issues
        - Security vulnerabilities
        - Maintainability concerns
```

**Problem Solving Loop**
```yaml
problem_solver:
  type: loop
  name: Problem Solving Loop
  description: Iteratively refines solutions
  max_iterations: 3
  instructions:
    - |
        Each iteration:
        1. Analyze the current solution
        2. Identify weaknesses or gaps
        3. Refine and enhance the approach
  sub_agents:
    - solution_refiner

solution_refiner:
  type: llm
  name: Solution Refiner
  model: gpt-4o
  instructions:
    - |
        Evaluate the solution critically and suggest improvements.
        Consider edge cases, efficiency, and robustness.
```

### Best Practices

1. **Reasonable Iterations**: 2-4 iterations typically sufficient
2. **Clear Progress**: Each iteration should meaningfully improve
3. **Convergence**: Instructions should guide toward refinement, not reimagining
4. **Final Polish**: Last iteration should produce final deliverable

### Limitations

- ❌ Cannot use `toolsets` directly (sub-agent can have tools)
- ❌ Must have exactly one sub-agent
- ❌ `max_iterations` must be between 1 and 10
- ⚠️ Cost increases linearly with iterations

## Sequence Agent

The **Sequence Agent** processes content through a defined sequential pipeline.

### Characteristics

- ✅ Sequential, ordered processing
- ✅ Each agent receives previous agent's output
- ✅ Clear workflow stages
- ✅ Specialized agents for each stage
- ⚠️ Cannot use tools directly
- ⚠️ Cannot be interactive

### Configuration

```yaml
agents:
  content_pipeline:
    type: sequence
    name: Content Creation Pipeline
    description: Research, write, and edit content sequentially
    instructions:
      - |
          Coordinate the content creation workflow:
          1. Research phase gathers information
          2. Writing phase creates draft
          3. Editing phase polishes final version
          
          Ensure each phase builds effectively on the previous.
    sub_agents:
      - researcher
      - writer
      - editor
  
  researcher:
    type: llm
    name: Researcher
    description: Gathers information and sources
    instructions:
      - |
          Research the topic thoroughly. Provide:
          - Key facts and statistics
          - Important concepts
          - Relevant sources
          - Suggested angles
    toolsets:
      - duckduckgo
  
  writer:
    type: llm
    name: Writer
    description: Creates content from research
    instructions:
      - |
          Using the research provided, write a comprehensive article.
          Create engaging content with clear structure and flow.
  
  editor:
    type: llm
    name: Editor
    description: Polishes and finalizes content
    instructions:
      - |
          Review and polish the draft. Ensure:
          - Grammar and spelling are perfect
          - Flow is smooth
          - Message is clear and impactful
          - Length is appropriate
```

### Key Properties

| Property | Required | Description |
|----------|----------|-------------|
| `type` | Yes | Must be `"sequence"` |
| `name` | Yes | Agent display name |
| `description` | Yes | Brief role description |
| `instructions` | Yes | Overall workflow coordination |
| `sub_agents` | Yes | Array of agents in execution order |
| `toolsets` | No | ⚠️ Not supported for sequence agents |

### Execution Flow

```
User Request
     │
     ▼
Agent 1 (Research)
     │
     ▼
Agent 2 (Write) ← receives Agent 1 output
     │
     ▼
Agent 3 (Edit) ← receives Agent 2 output
     │
     ▼
Final Result
```

### Use Cases

**Blog Post Production**
```yaml
blog_production:
  type: sequence
  name: Blog Production Pipeline
  description: Complete blog post creation workflow
  instructions:
    - |
        Create a publication-ready blog post through:
        1. Topic research and outline
        2. Draft creation
        3. SEO optimization
        4. Final editing
  sub_agents:
    - research_phase
    - draft_phase
    - seo_phase
    - editing_phase
```

**Report Generation**
```yaml
report_generator:
  type: sequence
  name: Report Generator
  description: Generates comprehensive business reports
  instructions:
    - |
        Generate a professional report:
        1. Collect and validate data
        2. Perform analysis
        3. Create visualizations
        4. Write executive summary
  sub_agents:
    - data_collector
    - data_analyst
    - visualization_creator
    - summary_writer
```

**Document Translation Pipeline**
```yaml
translation_pipeline:
  type: sequence
  name: Translation Pipeline
  description: Translates and localizes documents
  instructions:
    - |
        Complete translation workflow:
        1. Analyze source document
        2. Translate content
        3. Localize cultural references
        4. Quality review
  sub_agents:
    - source_analyzer
    - translator
    - localizer
    - qa_reviewer
```

**Customer Onboarding**
```yaml
onboarding_flow:
  type: sequence
  name: Customer Onboarding Flow
  description: Guides new customers through setup
  instructions:
    - |
        Onboard customers step-by-step:
        1. Collect requirements
        2. Configure account
        3. Setup integrations
        4. Provide training
  sub_agents:
    - requirements_collector
    - account_configurator
    - integration_specialist
    - trainer
```

### Best Practices

1. **Logical Order**: Arrange sub-agents in natural workflow sequence
2. **Clear Handoffs**: Each agent should produce output suitable for next agent
3. **Specialization**: Each stage agent should be focused and specialized
4. **Error Recovery**: Early stages should validate inputs thoroughly

### Limitations

- ❌ Cannot use `toolsets` directly (sub-agents can have tools)
- ❌ Cannot be set as `interactive: true`
- ❌ Execution is strictly sequential (no parallelism)
- ⚠️ Failure in one stage blocks subsequent stages

## Choosing the Right Type

### Decision Tree

```
Need real-time interaction?
├─ YES → LLM Agent
└─ NO → Need processing pattern?
         ├─ Multiple concurrent perspectives → Parallel Agent
         ├─ Iterative refinement → Loop Agent
         ├─ Sequential pipeline → Sequence Agent
         └─ Single-step processing → LLM Agent
```

### Comparison Matrix

| Requirement | LLM | Parallel | Loop | Sequence |
|-------------|-----|----------|------|----------|
| Interactive conversations | ✅ | ❌ | ❌ | ❌ |
| Direct tool access | ✅ | ❌ | ❌ | ❌ |
| Sub-agent delegation | ✅ | ✅ | ✅ | ✅ |
| Concurrent execution | ❌ | ✅ | ❌ | ❌ |
| Iterative refinement | ❌ | ❌ | ✅ | ❌ |
| Sequential workflow | ❌ | ❌ | ❌ | ✅ |
| Result combination | N/A | Auto | Iterative | Sequential |

### Use Case Mapping

| Scenario | Recommended Type | Why |
|----------|------------------|-----|
| Customer chat support | LLM Agent | Interactive, needs tools |
| Multi-expert analysis | Parallel Agent | Concurrent perspectives |
| Content refinement | Loop Agent | Iterative improvement |
| Content production | Sequence Agent | Sequential stages |
| Quick lookup | LLM Agent | Simple, single-step |
| Brainstorming | Parallel Agent | Multiple viewpoints |
| Problem solving | Loop Agent | Progressive refinement |
| Report generation | Sequence Agent | Collect → Analyze → Write |

## Combining Types

You can combine agent types within a single team for sophisticated workflows.

### Example: Research & Writing Team

```yaml
version: "1"
id: research-writing-team
name: Research & Writing Team
description: Comprehensive content creation with research and refinement
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Project Coordinator
    description: Manages overall workflow
    instructions:
      - |
          Coordinate content creation:
          1. Route research requests to research_pipeline
          2. Route writing requests to content_refiner
          3. Provide final deliverables to user
    sub_agents:
      - research_pipeline
      - content_refiner
  
  # Parallel Agent for multi-source research
  research_pipeline:
    type: parallel
    name: Research Pipeline
    description: Researches from multiple angles
    instructions:
      - |
          Combine all research into comprehensive brief with:
          - Facts and statistics
          - Expert opinions
          - Counterarguments
          - Source citations
    sub_agents:
      - academic_researcher
      - news_researcher
      - expert_researcher
  
  academic_researcher:
    type: llm
    name: Academic Researcher
    description: Finds academic sources
    toolsets:
      - duckduckgo
  
  news_researcher:
    type: llm
    name: News Researcher
    description: Finds recent news
    toolsets:
      - duckduckgo
  
  expert_researcher:
    type: llm
    name: Expert Researcher
    description: Finds expert opinions
    toolsets:
      - duckduckgo
  
  # Sequence Agent for structured writing
  content_refiner:
    type: sequence
    name: Content Refinement Pipeline
    description: Creates and polishes content
    instructions:
      - |
          Process content through:
          1. Drafting based on research
          2. Structural editing
          3. Final polish
    sub_agents:
      - drafter
      - structural_editor
      - copy_editor
  
  drafter:
    type: llm
    name: Draft Writer
    description: Creates initial draft
  
  structural_editor:
    type: llm
    name: Structural Editor
    description: Improves structure and flow
  
  copy_editor:
    type: llm
    name: Copy Editor
    description: Final polish and proofreading
```

## Performance Considerations

### Execution Time

| Type | Time Complexity | Notes |
|------|----------------|-------|
| LLM Agent | 1x | Single execution |
| Parallel Agent | ~1x | Concurrent (as slow as slowest sub-agent) |
| Loop Agent | N×1x | N = max_iterations |
| Sequence Agent | N×1x | N = number of sub-agents |

### Cost Implications

- **Parallel**: Cost of all sub-agents, but faster wall-clock time
- **Loop**: Linear cost increase with iterations
- **Sequence**: Linear cost increase with pipeline length
- **LLM**: Single execution cost

### Optimization Tips

1. **Parallel**: Keep sub-agents similar in complexity
2. **Loop**: Use 2-3 iterations typically (more = diminishing returns)
3. **Sequence**: Minimize pipeline length when possible
4. **General**: Cache results when appropriate

## Troubleshooting

### Common Issues

**Parallel Agent Not Working**
- ✅ Check: Have you defined sub-agents?
- ✅ Check: Are sub-agents independent (no communication needed)?
- ✅ Check: Did you add instructions for combining results?

**Loop Agent Not Improving**
- ✅ Check: Are instructions guiding refinement?
- ✅ Check: Is max_iterations reasonable (2-4)?
- ✅ Check: Can the sub-agent access necessary context?

**Sequence Agent Failing**
- ✅ Check: Are sub-agents in correct order?
- ✅ Check: Does each agent produce suitable input for the next?
- ✅ Check: Are early stages validating inputs?

**Type Not Recognized**
- ✅ Check: Did you spell type correctly? (`"llm"`, `"parallel"`, `"loop"`, `"sequence"`)
- ✅ Check: Is the type in quotes?
- ✅ Check: Is the YAML syntax valid?

## Next Steps

- **[Creating Agents](creating-agents.md)** - Build your first team
- **[Configuration](configuration.md)** - Complete reference
- **[Examples](examples.md)** - Real-world templates
- **[Multi-Agent Workflows](../advanced/multi-agent.md)** - Complex patterns

---

**Related:**  
[Overview](overview.md) | [Creating Teams](creating-agents.md) | [Examples](examples.md)
