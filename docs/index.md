# MyAgents User Guide

Welcome to the **MyAgents User Guide**! This comprehensive documentation will help you create, configure, and deploy powerful AI agent teams with integrated toolsets.

## What is MyAgents?

MyAgents is a sophisticated AI agent platform that enables you to build **teams of specialized AI agents** for specific business workflows and domains. Unlike general-purpose AI, MyAgents allows you to create agent teams with:

- **Domain expertise** - Specialized knowledge and context
- **Team coordination** - Multiple agents working together
- **Tool integration** - Access to external systems via MCP (Model Context Protocol)
- **Configurable behavior** - Runtime customization through settings

## Key Concepts

### Teams of Agents

Each YAML file defines a **Team of Agents**:

- **Team** = Collection of agents working together
- **Agent** = Individual AI assistant with specific role
- **Coordination** = Agents delegate work to each other

### Agent Types

MyAgents supports four agent types:

- **LLM Agent** - Interactive AI assistant (default)
- **Parallel Agent** - Run multiple agents concurrently
- **Loop Agent** - Execute agent iteratively
- **Sequence Agent** - Sequential pipeline processing

## Quick Example

Here's a simple team with one agent that helps with grocery shopping:

```yaml
version: "1"
id: grocery-assistant
name: Grocery Shopping Assistant
description: Assists customers with grocery shopping and product recommendations
interactive: true
default_agent: shopping_helper

agents:
  shopping_helper:
    type: llm
    name: Shopping Helper
    description: Helps customers find products
    instructions:
      - |
        You are a friendly grocery shopping assistant. 
        Help customers find products, provide recommendations, 
        and answer questions about ingredients and recipes.
    toolsets:
      - duckduckgo
```

## Documentation Structure

### [Getting Started](getting-started.md)
Quick start guide to create your first team in minutes.

### [Agents](agents/overview.md)
Learn about agent teams, types, and coordination patterns.

- [Agent Types](agents/agent-types.md) - LLM, Parallel, Loop, and Sequence agents
- [Creating Teams](agents/creating-agents.md) - Step-by-step team creation
- [Configuration](agents/configuration.md) - Complete configuration reference
- [Examples](agents/examples.md) - Real-world team examples

### [Tools](tools/overview.md)
Discover and integrate MCP tools into your agents.

- [Available Tools](tools/available-tools.md) - Browse available toolsets
- [Using Tools](tools/using-tools.md) - Integrate tools in your agents

### [Advanced Topics](advanced/settings.md)
Master advanced features for production deployments.

- [Settings & Configuration](advanced/settings.md) - Runtime configuration
- [Conditional Instructions](advanced/conditional-instructions.md) - Dynamic behavior
- [Multi-Agent Workflows](advanced/multi-agent.md) - Complex team orchestration

### [Reference](reference/schema.md)
Technical reference documentation.

- [YAML Schema](reference/schema.md) - Complete schema documentation

## Common Use Cases

### 🛒 E-Commerce
Create shopping assistant teams that help customers find products, compare prices, and make purchase decisions.

### 📝 Content Production
Build content creation teams with researchers, writers, and editors working together.

### 📊 Business Analytics
Generate reports with teams that collect data, analyze it, and format professional outputs.

### 🎯 Customer Support
Route support tickets to specialized agents based on issue type and urgency.

### 📅 Family Management
Coordinate meal planning, calendars, household tasks, and family activities with a team of agents.

## Key Features

### 🤖 Multi-Agent Teams
Build complex teams with specialized agents that work together to handle sophisticated workflows.

### 🛠️ MCP Tool Integration
Seamlessly integrate external tools and services using the Model Context Protocol standard.

### ⚙️ Dynamic Configuration
Configure team behavior at runtime using settings and conditional instructions.

### 🔄 Agent Types
Use LLM, Parallel, Loop, and Sequence agents to create powerful workflows.

### 📊 Production-Ready
Enterprise-grade architecture with DynamoDB storage, comprehensive logging, and error handling.

## Getting Help

### Documentation
This user guide covers everything from basic concepts to advanced patterns.

### Examples
Check the [Examples](agents/examples.md) section for real-world team templates.

### Support
For technical issues or questions, contact your system administrator.

## What's Next?

Ready to create your first team? Head over to the [Getting Started](getting-started.md) guide!

---

**Version:** 1.0  
**Last Updated:** 2025-10-25  
**Platform:** MyAgents AI Agent Platform
