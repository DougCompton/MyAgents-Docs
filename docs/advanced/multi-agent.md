# Multi-Agent Workflows

Complex workflows using multiple specialized agents working together.

## Overview

**Multi-agent workflows** enable:

- 🎯 **Specialization** - Each agent excels at specific tasks
- 🔄 **Collaboration** - Agents delegate work to each other
- 📊 **Parallel Processing** - Multiple agents execute concurrently
- 🔁 **Sequential Pipelines** - Ordered processing through stages
- 🌳 **Hierarchical Organization** - Multi-level coordination

## Sub-Agent Discovery

When an LLM agent has `sub_agents` defined, the system **automatically provides** information about available sub-agents in the agent's instructions. This happens through the `<SubAgents />` data tag, which is auto-appended at runtime.

### Automatic Behavior

You don't need to manually add the `<SubAgents />` tag to your instructions. The system automatically:

1. **Detects** when an agent has the `sub_agents` property
2. **Appends** the `<SubAgents />` data tag to the agent's instructions
3. **Expands** the tag at runtime with detailed information about each sub-agent

### What Gets Included

The expanded `<SubAgents />` tag provides:

- **Name** - The sub-agent's identifier
- **Description** - What the sub-agent does
- **Instructions** (preview) - First 200 characters of the sub-agent's instructions

This information helps the coordinator agent understand which sub-agent to delegate to.

### Example

Given this agent definition:

```yaml
agents:
  coordinator:
    name: coordinator
    description: Routes requests to specialists
    instructions:
      - |
        You are a coordinator. Analyze requests and delegate to the appropriate specialist.
    sub_agents:
      - technical_support
      - billing_support

  technical_support:
    name: technical_support
    description: Handles technical issues
    instructions:
      - |
        Diagnose and resolve technical problems.
        Provide step-by-step solutions.

  billing_support:
    name: billing_support
    description: Handles billing inquiries
    instructions:
      - |
        Address billing questions and process refunds.
```

The coordinator agent automatically receives expanded instructions like:

```
You are a coordinator. Analyze requests and delegate to the appropriate specialist.

Available Sub-Agents:
- technical_support: Handles technical issues
  Instructions: Diagnose and resolve technical problems. Provide step-by-step solutions.

- billing_support: Handles billing inquiries
  Instructions: Address billing questions and process refunds.
```

### Manual Override

If you want to customize the sub-agent information, you can manually include `<SubAgents />` in your instructions. The system will detect the existing tag and won't duplicate it.

```yaml
coordinator:
  instructions:
    - |
      You are a coordinator.

      <SubAgents />

      Use the transfer_task tool to delegate work.
  sub_agents:
    - specialist_a
    - specialist_b
```

---

## Sharing Data Between Agents

Agents can pass their outputs to other agents using the `output_key` property. This enables sophisticated workflows where agents build on each other's work.

### How It Works

1. **Agent Saves Output:** An agent with `output_key` saves its response for later use
2. **Other Agents Reference:** Other agents use `{keyName}` placeholders in their instructions to access saved outputs
3. **Automatic Replacement:** Placeholders are replaced with the actual content before the agent runs

### Basic Example

```yaml
version: "1"
id: research-writing
name: Research & Writing Team
description: Research followed by content creation
interactive: true
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Coordinator
    description: Manages workflow
    instructions:
      - |
        First, transfer_task to researcher.
        After research completes, transfer_task to writer.
    sub_agents:
      - researcher
      - writer

  researcher:
    type: llm
    name: Researcher
    description: Conducts research
    output_key: research_summary    # Save output for later
    instructions:
      - |
        Research the topic and provide a comprehensive summary
        with key facts, statistics, and sources.
    toolsets:
      - duckduckgo

  writer:
    type: llm
    name: Writer
    description: Creates content from research
    instructions:
      - |
        Write an article based on this research:

        {research_summary}    # Use researcher's output

        Create engaging, well-structured content.
```

**Execution Flow:**

1. Coordinator delegates to researcher
2. Researcher completes and saves its output
3. Coordinator delegates to writer
4. Writer receives the research content and creates the article

### Use Case: Multi-Stage Pipeline

Agents can build on each other's work through multiple stages:

```yaml
agents:
  stage_1_collector:
    type: llm
    output_key: raw_data
    instructions:
      - |
        Collect data on the topic.
        Output format: Bullet points with sources.

  stage_2_analyzer:
    type: llm
    output_key: analysis
    instructions:
      - |
        Analyze this data:
        {raw_data}

        Identify trends, insights, and key findings.

  stage_3_visualizer:
    type: llm
    output_key: chart_recommendations
    instructions:
      - |
        Based on this analysis:
        {analysis}

        Recommend visualizations (charts, graphs, tables).

  stage_4_writer:
    type: llm
    instructions:
      - |
        Write a comprehensive report using:

        Data: {raw_data}
        Analysis: {analysis}
        Visualizations: {chart_recommendations}
```

### Use Case: Parallel Collection + Synthesis

Multiple agents work in parallel, then a coordinator synthesizes:

```yaml
agents:
  coordinator:
    type: llm
    name: Synthesis Coordinator
    instructions:
      - |
        First, use parallel_researchers to gather information.
        Then synthesize all findings into a comprehensive report.
    sub_agents:
      - parallel_researchers

  parallel_researchers:
    type: parallel
    name: Multi-Source Researchers
    instructions:
      - |
        Combine all research perspectives:

        Academic: {academic_findings}
        Industry: {industry_findings}
        News: {news_findings}

        Create a unified research brief.
    sub_agents:
      - academic_researcher
      - industry_researcher
      - news_researcher

  academic_researcher:
    type: llm
    output_key: academic_findings
    instructions:
      - |
        Research academic sources and scholarly papers.
    toolsets:
      - duckduckgo

  industry_researcher:
    type: llm
    output_key: industry_findings
    instructions:
      - |
        Research industry reports and expert analysis.
    toolsets:
      - duckduckgo

  news_researcher:
    type: llm
    output_key: news_findings
    instructions:
      - |
        Research recent news and current trends.
    toolsets:
      - duckduckgo
```

### Use Case: Iterative Refinement with Context

Loop agents can reference initial input and progressively improve:

```yaml
agents:
  initial_writer:
    type: llm
    output_key: draft_v1
    instructions:
      - |
        Write a first draft on the topic.

  refinement_loop:
    type: loop
    max_iterations: 3
    name: Content Refiner
    instructions:
      - |
        Iteratively improve the content.
        Original draft: {draft_v1}

        Each iteration should enhance clarity and quality.
    sub_agents:
      - content_improver

  content_improver:
    type: llm
    output_key: draft_v1    # Overwrites each iteration
    instructions:
      - |
        Improve this content:
        {draft_v1}

        Focus on clarity, flow, and impact.
```

### Important Details

**When Saved Outputs Are Available:**
- ✅ Available to all agents during the same conversation turn
- ❌ NOT saved between different chat messages
- ❌ NOT shared between different users

**Using Placeholders:**
- `{keyName}` → Replaced with the saved output
- `{missing_key}` → Left as-is if the key doesn't exist
- Whitespace is ignored: `{  keyName  }` works the same as `{keyName}`

**Naming Your Keys:**
- Use descriptive names: `research_summary`, `analysis_results`
- Snake_case recommended: `user_preferences`, `raw_data`
- Dashes and dots also work: `key-with-dashes`, `key.with.dots`

### Best Practices

**1. Use Descriptive Keys**

```yaml
# ✅ Good - Clear purpose
output_key: research_findings
output_key: technical_analysis
output_key: user_preferences

# ❌ Bad - Unclear
output_key: data
output_key: result
output_key: output
```

**2. Document State Dependencies**

```yaml
agents:
  analyzer:
    instructions:
      - |
        # This agent requires 'research_findings' from researcher
        Analyze these findings:
        {research_findings}
```

**3. Provide Fallback Content**

```yaml
agents:
  writer:
    instructions:
      - |
        Research findings:
        {research_findings}

        If no research is available above, conduct your own research first.
```

**4. Use Sequential Types for Dependencies**

When agents depend on each other's outputs, use `sequence` type:

```yaml
pipeline:
  type: sequence
  sub_agents:
    - collector    # output_key: raw_data
    - analyzer     # uses {raw_data}
    - reporter     # uses {raw_data} and {analysis}
```

### Common Patterns

**Pattern 1: Research → Write**
```yaml
researcher: output_key: research
writer: uses {research}
```

**Pattern 2: Collect → Analyze → Report**
```yaml
collector: output_key: data
analyzer: uses {data}, output_key: insights
reporter: uses {data} and {insights}
```

**Pattern 3: Parallel Gather → Synthesize**
```yaml
agent_a: output_key: findings_a
agent_b: output_key: findings_b
agent_c: output_key: findings_c
synthesizer: uses {findings_a}, {findings_b}, {findings_c}
```

**Pattern 4: Draft → Iterate → Finalize**
```yaml
drafter: output_key: draft
refiner_loop: uses {draft}, overwrites output_key: draft
finalizer: uses {draft}
```

### Troubleshooting

**Placeholder not replaced (shows `{keyName}` in output):**

Check:
- ✅ The earlier agent has `output_key` defined
- ✅ The earlier agent completed successfully
- ✅ Key name matches exactly (case-sensitive)
- ✅ Agents ran in the correct order

**Missing or incorrect data:**

Verify:
- ✅ The coordinator delegates to agents in the right sequence
- ✅ The agent that saves the output ran before the agent that uses it
- ✅ No typos in key names

**Data not available in next conversation:**

Remember:
- Saved outputs only last for one conversation turn
- For data that needs to persist between conversations, use the `memory-server` toolset

---

## Team Patterns

### Hub-and-Spoke

One coordinator routes requests to specialized agents.

**Structure:**
```
      Coordinator
      /    |    \
     /     |     \
  Agent  Agent  Agent
    A      B      C
```

**When to use:**
- Multiple specialist domains
- Clear routing logic
- Independent specialist tasks

**Example:**
```yaml
version: "1"
id: customer-support
name: Customer Support Team
description: Routes support tickets to appropriate specialists
interactive: true
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Support Coordinator
    description: Routes tickets based on issue type
    instructions:
      - |
        Analyze the customer's issue and route:
          - Technical problems → transfer_task to technical_support
          - Billing questions → transfer_task to billing_support
          - Account issues → transfer_task to account_support
          
          Provide context when delegating.
    sub_agents:
      - technical_support
      - billing_support
      - account_support
  
  technical_support:
    type: llm
    name: Technical Support Specialist
    description: Handles technical issues
    instructions:
      - |
        Diagnose and resolve technical problems.
          Provide step-by-step solutions with documentation links.
    toolsets:
      - memory-server
  
  billing_support:
    type: llm
    name: Billing Support Specialist
    description: Handles billing inquiries
    instructions:
      - |
        Address billing questions, process refunds, update payment methods.
          Always verify account ownership first.
    toolsets:
      - memory-server
  
  account_support:
    type: llm
    name: Account Support Specialist
    description: Handles account management
    instructions:
      - |
        Assist with passwords, settings, security, and account deletion.
          Security-first approach - verify identity before changes.
    toolsets:
      - memory-server
```

**Benefits:**
- ✅ Clear separation of concerns
- ✅ Easy to add new specialists
- ✅ Simple routing logic

**Trade-offs:**
- ⚠️ Coordinator becomes bottleneck
- ⚠️ No direct communication between specialists

---

### Hierarchical Delegation

Multi-level coordination with teams managing teams.

**Structure:**
```
    Project Manager
          |
    Team Lead
      /      \
  Agent A   Agent B
```

**When to use:**
- Complex projects with sub-projects
- Multiple levels of expertise
- Workflow requires coordination at different levels

**Example:**
```yaml
version: "1"
id: content-production
name: Content Production Team
description: Manages multi-stage content creation with research, writing, and editing teams
interactive: true
default_agent: project_manager

agents:
  project_manager:
    type: llm
    name: Project Manager
    description: Oversees entire content production
    instructions:
      - |
          Coordinate content creation:
          1. Understand requirements
          2. Delegate to research_team_lead
          3. Then delegate to writing_team_lead with research
          4. Review final output
    sub_agents:
      - research_team_lead
      - writing_team_lead
  
  research_team_lead:
    type: llm
    name: Research Team Lead
    description: Coordinates research efforts
    instructions:
      - |
          Manage research phase:
          - Assign specific research angles to specialists
          - Synthesize findings
          - Create research brief
    sub_agents:
      - academic_researcher
      - news_researcher
      - expert_researcher
  
  academic_researcher:
    type: llm
    name: Academic Researcher
    description: Researches academic sources
    toolsets:
      - duckduckgo
  
  news_researcher:
    type: llm
    name: News Researcher
    description: Researches current news
    toolsets:
      - duckduckgo
  
  expert_researcher:
    type: llm
    name: Expert Researcher
    description: Researches expert opinions
    toolsets:
      - duckduckgo
  
  writing_team_lead:
    type: llm
    name: Writing Team Lead
    description: Coordinates writing and editing
    instructions:
      - |
          Manage writing phase:
          1. Have writer create draft from research
          2. Have editor polish the draft
          3. Deliver final version
    sub_agents:
      - content_writer
      - copy_editor
  
  content_writer:
    type: llm
    name: Content Writer
    description: Writes content
    model: anthropic
  
  copy_editor:
    type: llm
    name: Copy Editor
    description: Edits and polishes
    model: anthropic
```

**Benefits:**
- ✅ Scales to complex workflows
- ✅ Clear chain of command
- ✅ Specialized team coordination

**Trade-offs:**
- ⚠️ More complexity to manage
- ⚠️ Longer execution paths
- ⚠️ Coordination overhead

---

### Parallel Processing

Multiple agents execute simultaneously.

**Structure:**
```
  Parallel Coordinator
    /    |    \
   /     |     \
Agent   Agent  Agent
  A       B      C
  
(All execute concurrently)
```

**When to use:**
- Independent tasks
- Multi-perspective analysis
- Batch processing
- Time-sensitive workflows

**Example:**
```yaml
version: "1"
id: product-analyzer
name: Product Analysis Team
description: Analyzes products from multiple perspectives simultaneously
default_agent: comprehensive_analyzer

agents:
  comprehensive_analyzer:
    type: parallel
    name: Multi-Perspective Analyzer
    description: Coordinates parallel analysis
    instructions:
      - |
          Synthesize insights from all perspectives:
          
          1. **Overview** - Combined assessment
          2. **Technical Analysis** - From technical_analyst
          3. **Business Analysis** - From business_analyst
          4. **User Analysis** - From user_analyst
          5. **Recommendations** - Based on all perspectives
    sub_agents:
      - technical_analyst
      - business_analyst
      - user_analyst
  
  technical_analyst:
    type: llm
    name: Technical Analyst
    description: Technical feasibility assessment
    instructions:
      - |
          Analyze technical aspects:
          - Implementation complexity
          - Scalability
          - Security considerations
          - Technical risks
          - Maintenance requirements
  
  business_analyst:
    type: llm
    name: Business Analyst
    description: Business value assessment
    instructions:
      - |
          Analyze business aspects:
          - Market opportunity
          - ROI potential
          - Competitive advantage
          - Resource requirements
          - Business risks
  
  user_analyst:
    type: llm
    name: User Experience Analyst
    description: User impact assessment
    instructions:
      - |
          Analyze user aspects:
          - User value
          - Usability
          - Accessibility
          - Adoption barriers
          - User satisfaction potential
```

**Benefits:**
- ✅ Faster execution (concurrent)
- ✅ Multiple perspectives
- ✅ Independent analysis

**Trade-offs:**
- ⚠️ Cannot use tools in parallel agent (only sub-agents)
- ⚠️ Results must be synthesized
- ⚠️ No communication between executing agents

---

### Sequential Pipeline

Ordered processing through defined stages.

**Structure:**
```
Stage 1 → Stage 2 → Stage 3 → Stage 4
```

**When to use:**
- Multi-stage processes
- Each stage depends on previous
- Quality gates between stages
- Progressive refinement

**Example:**
```yaml
version: "1"
id: report-generator
name: Report Generation Team
description: Generates business reports through structured pipeline
default_agent: report_pipeline

agents:
  report_pipeline:
    type: sequence
    name: Report Generation Pipeline
    description: Sequential report creation
    instructions:
      - |
          Generate comprehensive report through stages:
          1. Data collection and validation
          2. Statistical analysis
          3. Visualization recommendations
          4. Executive summary writing
    sub_agents:
      - data_collector
      - data_analyst
      - visualization_specialist
      - summary_writer
  
  data_collector:
    type: llm
    name: Data Collector
    description: Collects and validates data
    instructions:
      - |
          Stage 1: Data Collection
          - Gather required data
          - Validate completeness
          - Check for anomalies
          - Organize for analysis
  
  data_analyst:
    type: llm
    name: Data Analyst
    description: Analyzes data statistically
    instructions:
      - |
          Stage 2: Analysis
          - Calculate key metrics
          - Identify trends
          - Perform statistical tests
          - Generate insights
  
  visualization_specialist:
    type: llm
    name: Visualization Specialist
    description: Designs visualizations
    instructions:
      - |
          Stage 3: Visualizations
          - Recommend chart types
          - Design visual layouts
          - Specify data presentation
          - Highlight key findings
  
  summary_writer:
    type: llm
    name: Executive Summary Writer
    description: Writes executive summary
    model: anthropic
    instructions:
      - |
          Stage 4: Summary
          - Write executive overview
          - Highlight key findings
          - Provide recommendations
          - Include next steps
```

**Benefits:**
- ✅ Clear workflow stages
- ✅ Quality control at each step
- ✅ Each stage specialized

**Trade-offs:**
- ⚠️ Sequential (not parallel)
- ⚠️ Failure in one stage blocks later stages
- ⚠️ Cannot use tools in sequence agent (only sub-agents)

---

### Loop Refinement

Iterative improvement through repeated execution.

**Structure:**
```
Input → Agent → Output → Agent → Output → Agent → Final
       (Iteration 1)    (Iteration 2)    (Iteration 3)
```

**When to use:**
- Content refinement
- Progressive improvement
- Quality iteration
- Problem solving

**Example:**
```yaml
version: "1"
id: content-refiner
name: Content Refinement Team
description: Iteratively refines content to perfection
default_agent: refinement_loop

agents:
  refinement_loop:
    type: loop
    name: Content Refinement Loop
    max_iterations: 3
    description: Iteratively improves content
    instructions:
      - |
          Refinement strategy:
          - Iteration 1: Structure and organization
          - Iteration 2: Clarity and language
          - Iteration 3: Style and polish
          
          Each iteration builds on the previous.
    sub_agents:
      - content_improver
  
  content_improver:
    type: llm
    name: Content Improver
    description: Improves content each iteration
    model: anthropic
    instructions:
      - |
          Improve the content:
          - Fix issues from previous iteration
          - Enhance clarity and flow
          - Strengthen arguments
          - Polish language
          - Maintain core message
```

**Benefits:**
- ✅ Progressive improvement
- ✅ Quality refinement
- ✅ Converges to better output

**Trade-offs:**
- ⚠️ Cost increases linearly with iterations
- ⚠️ Diminishing returns after 3-4 iterations
- ⚠️ Must have exactly 1 sub-agent

---

## Combining Patterns

Mix patterns for sophisticated workflows.

### Parallel + Sequential

Parallel research feeding into sequential writing:

```yaml
version: "1"
id: research-writing-team
name: Research & Writing Team
description: Parallel research followed by sequential content creation
interactive: true
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Project Coordinator
    description: Coordinates overall workflow
    instructions:
      - |
          Two-phase workflow:
          1. Parallel research phase → multi_source_researcher
          2. Sequential writing phase → content_pipeline
    sub_agents:
      - multi_source_researcher
      - content_pipeline
  
  # PARALLEL: Research from multiple sources
  multi_source_researcher:
    type: parallel
    name: Multi-Source Researcher
    description: Researches from multiple angles
    instructions:
      - |
          Synthesize research from all sources:
          - Academic findings
          - News and trends
          - Expert opinions
          - Create comprehensive research brief
    sub_agents:
      - academic_researcher
      - news_researcher
      - expert_researcher
  
  academic_researcher:
    type: llm
    name: Academic Researcher
    toolsets:
      - duckduckgo
  
  news_researcher:
    type: llm
    name: News Researcher
    toolsets:
      - duckduckgo
  
  expert_researcher:
    type: llm
    name: Expert Researcher
    toolsets:
      - duckduckgo
  
  # SEQUENTIAL: Writing pipeline
  content_pipeline:
    type: sequence
    name: Content Creation Pipeline
    description: Sequential content creation
    instructions:
      - |
          Create content through stages:
          1. Draft from research
          2. SEO optimization
          3. Final editing
    sub_agents:
      - draft_writer
      - seo_optimizer
      - copy_editor
  
  draft_writer:
    type: llm
    name: Draft Writer
    model: anthropic
  
  seo_optimizer:
    type: llm
    name: SEO Optimizer
  
  copy_editor:
    type: llm
    name: Copy Editor
    model: anthropic
```

### Hub-and-Spoke + Loop

Coordinator routes to specialists, including a refinement loop:

```yaml
version: "1"
id: content-team
name: Content Team
description: Routes content requests with iterative refinement option
interactive: true
default_agent: coordinator

agents:
  coordinator:
    type: llm
    name: Content Coordinator
    description: Routes content requests
    instructions:
      - |
          Route based on request:
          - New content → quick_writer
          - Existing content to improve → refinement_loop
          - Research needed → researcher
    sub_agents:
      - quick_writer
      - refinement_loop
      - researcher
  
  quick_writer:
    type: llm
    name: Quick Writer
    description: Creates new content quickly
    model: anthropic
  
  refinement_loop:
    type: loop
    max_iterations: 3
    name: Content Refinement Loop
    description: Iteratively improves existing content
    instructions:
      - |
          Progressive refinement over 3 iterations
    sub_agents:
      - content_polisher
  
  content_polisher:
    type: llm
    name: Content Polisher
    model: anthropic
  
  researcher:
    type: llm
    name: Researcher
    toolsets:
      - duckduckgo
```

## Workflow Design Best Practices

### Keep It Simple

Start with simple patterns and add complexity only when needed.

**Good - Start simple:**
```yaml
# Single coordinator with specialists
agents:
  coordinator:
    sub_agents:
      - specialist_a
      - specialist_b
```

**Bad - Overly complex:**
```yaml
# Unnecessary hierarchy
agents:
  top_coordinator:
    sub_agents:
      - mid_coordinator_1
        sub_agents:
          - low_coordinator_1
            sub_agents:
              - specialist
```

### Clear Responsibilities

Each agent should have clear, focused responsibility.

**Good:**
```yaml
technical_support:
  description: Handles technical issues and bugs
  
billing_support:
  description: Handles billing and payment questions
```

**Bad:**
```yaml
general_support:
  description: Handles everything
```

### Minimize Delegation Depth

Avoid deep delegation chains.

**Good - 2 levels:**
```
Coordinator → Specialist
```

**Acceptable - 3 levels:**
```
Manager → Team Lead → Specialist
```

**Bad - Too deep:**
```
Executive → Manager → Lead → Coordinator → Specialist
```

### Use Appropriate Types

Choose the right agent type for the workflow.

**Parallel for independent tasks:**
```yaml
# Good - Independent analysis
multi_analyst:
  type: parallel
  sub_agents:
    - technical_analyst
    - business_analyst
    - user_analyst
```

**Sequential for dependent stages:**
```yaml
# Good - Each stage needs previous output
pipeline:
  type: sequence
  sub_agents:
    - stage_1_collector
    - stage_2_processor
    - stage_3_formatter
```

**Loop for refinement:**
```yaml
# Good - Iterative improvement
refiner:
  type: loop
  max_iterations: 3
  sub_agents:
    - content_improver
```

## Common Patterns

### Content Creation Factory

```yaml
# Parallel research → Sequential creation → Loop refinement
researcher (parallel)
  → writer (sequence)
    → refiner (loop)
```

### Customer Service Router

```yaml
# Hub-and-spoke with specialized agents
coordinator
  ├── technical_support
  ├── billing_support
  └── account_support
```

### Data Processing Pipeline

```yaml
# Sequential stages
collector → validator → analyzer → reporter
```

### Multi-Perspective Analyzer

```yaml
# Parallel analysis
analyzer (parallel)
  ├── perspective_1
  ├── perspective_2
  └── perspective_3
```

## Debugging Multi-Agent Workflows

### Trace Execution Flow

Use clear logging in instructions:

```yaml
coordinator:
  instructions:
    - |
        [COORDINATOR] Analyzing request type...
        [COORDINATOR] Routing to technical_support
        
technical_support:
  instructions:
    - |
        [TECHNICAL_SUPPORT] Received request from coordinator
        [TECHNICAL_SUPPORT] Processing technical issue...
```

### Test Individual Agents

Test each agent independently before integration:

```yaml
# Test agent in isolation
test_technical_support:
  # Same as technical_support but standalone
  # Test with sample inputs
```

### Validate Delegation Logic

Ensure routing logic is clear and correct:

```yaml
coordinator:
  instructions:
    - |
          Routing logic:
          - "error" OR "bug" → technical_support
          - "payment" OR "invoice" OR "refund" → billing_support
          - "password" OR "login" OR "account" → account_support
          
          Log routing decision for debugging.
```

## Performance Considerations

### Execution Time

| Pattern | Time Complexity | Notes |
|---------|----------------|-------|
| Hub-and-Spoke | 2× (coordinator + specialist) | Simple routing overhead |
| Hierarchical | 3-4× (multiple levels) | Each level adds overhead |
| Parallel | ~1× (as slow as slowest) | Concurrent execution |
| Sequential | N× (N = stages) | Each stage is sequential |
| Loop | N× (N = iterations) | Linear with iterations |

### Cost Optimization

**Parallel processing:**
- Higher total LLM calls
- Lower wall-clock time
- Good for time-sensitive workflows

**Sequential processing:**
- Same total LLM calls as parallel
- Higher wall-clock time
- Good for dependent stages

**Loop refinement:**
- Cost increases with iterations
- Keep iterations to 2-4
- Diminishing returns after that

## Next Steps

- **[Agent Types](../agents/agent-types.md)** - Understanding LLM, Parallel, Loop, Sequence
- **[Agent Examples](../agents/examples.md)** - Complete team templates
- **[Creating Teams](../agents/creating-agents.md)** - Building teams
- **[Settings](settings.md)** - Runtime configuration
- **[Conditional Instructions](conditional-instructions.md)** - Dynamic behavior

---

**Related:**  
[Agent Types](../agents/agent-types.md) | [Agent Examples](../agents/examples.md) | [Configuration](../agents/configuration.md)

