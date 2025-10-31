# Team Examples

Real-world agent team templates ready to customize and deploy.

## Simple Teams

### Weather Assistant

Single-agent team for weather information.

```yaml
version: "1"
id: weather-team
name: Weather Team
description: Provides current weather conditions, forecasts, and severe weather alerts
default_agent: weather_helper

agents:
  weather_helper:
    type: llm
    name: Weather Helper
    description: Retrieves weather information for any location
    instructions:
      - |
        You provide weather information using the weather tool.
          
          For current conditions:
          - Temperature and feels-like
          - Conditions (sunny, cloudy, rainy, etc.)
          - Humidity and wind speed
          - Visibility
          
          For forecasts:
          - Daily high/low temperatures
          - Precipitation chances
          - General conditions
          
          For severe weather:
          - Alert type and severity
          - Affected areas
          - Duration and recommendations
          
          Always include the location and timestamp.
    toolsets:
      - weather-tool
```

**Use cases:**
- Quick weather checks
- Travel planning
- Outdoor activity planning
- Severe weather monitoring

---

### Calculator

Simple computation assistant.

```yaml
version: "1"
id: calculator-team
name: Calculator Team
description: Performs mathematical calculations, unit conversions, and basic statistics
default_agent: calculator

agents:
  calculator:
    type: llm
    name: Calculator
    description: Performs mathematical operations
    instructions:
      - |
        You perform mathematical calculations accurately.
          
          Capabilities:
          - Basic arithmetic (addition, subtraction, multiplication, division)
          - Advanced math (exponents, roots, logarithms)
          - Unit conversions (length, weight, volume, temperature)
          - Percentage calculations
          - Basic statistics (mean, median, mode, standard deviation)
          
          Always:
          - Show your work
          - Explain the formula used
          - Round to appropriate precision
          - Include units in results
```

**Use cases:**
- Quick calculations
- Unit conversions
- Financial calculations
- Statistical analysis

---

## E-Commerce Teams

### Shopping Assistant

Interactive shopping experience with meal planning.

```yaml
version: "1"
id: shopping-team
name: Shopping Team
description: Assists with grocery shopping, meal planning, product recommendations, and shopping lists
interactive: true
default_agent: shopping_coordinator

settings:
  - name: dietary_restrictions
    type: string
    title: Dietary Restrictions
    description: Filter products by dietary needs
    defaultValue: none

  - name: budget_conscious
    type: bool
    title: Budget Mode
    description: Prioritize value and deals
    defaultValue: false

agents:
  shopping_coordinator:
    type: llm
    name: Shopping Coordinator
    description: Routes shopping requests to specialists
    instructions:
      - |
        Welcome customers and route their requests:
          
          - Product searches → transfer_task to product_specialist
          - Meal planning → transfer_task to meal_planner
          - Price comparisons → transfer_task to price_analyst
          - Recipe questions → transfer_task to recipe_specialist
          
          Maintain a friendly, helpful tone throughout.
    sub_agents:
      - product_specialist
      - meal_planner
      - price_analyst
      - recipe_specialist
  
  product_specialist:
    type: llm
    name: Product Specialist
    description: Finds and recommends products
    instructions:
      - if: "settings.dietary_restrictions != \"none\""
        content: |
          Filter all products by dietary restriction: {settings.dietary_restrictions}
          Clearly label dietary attributes on recommendations.

      - if: "settings.budget_conscious"
        content: |
          Prioritize value and deals:
          - Sort by price per unit
          - Highlight sale items
          - Suggest generic brands
          - Mention bulk discounts

      - |
        Search the product catalog for items matching customer needs.
          
          Provide 3-5 recommendations with:
          - Product name and brand
          - Price and size (include unit price)
          - Customer rating (if 4+ stars, highlight it)
          - Key features or benefits
          - Current availability
          
          Ask clarifying questions if needs are unclear.
    toolsets:
      - product-catalog
      - memory-server
  
  meal_planner:
    type: llm
    name: Meal Planner
    description: Creates meal plans and shopping lists
    instructions:
      - if: "settings.dietary_restrictions != \"none\""
        content: |
          Create meal plans for: {settings.dietary_restrictions}
          Ensure all ingredients and recipes comply with this restriction.

      - |
        Create meal plans based on:
          - Number of servings needed
          - Number of meals (breakfast, lunch, dinner, snacks)
          - Dietary preferences
          - Cuisine preferences
          - Time constraints (quick meals vs elaborate)
          
          For each meal provide:
          - Recipe name
          - Ingredients list with quantities
          - Estimated prep/cook time
          - Nutritional highlights
          
          Generate consolidated shopping list organized by category:
          - Produce
          - Meat & Seafood
          - Dairy & Eggs
          - Pantry staples
          - Other
    toolsets:
      - memory-server
  
  price_analyst:
    type: llm
    name: Price Analyst
    description: Compares prices and finds deals
    instructions:
      - |
        Compare prices across:
          - Different brands
          - Different sizes/quantities
          - Generic vs name brand
          
          Highlight:
          - Best value (lowest price per unit)
          - Current sales and discounts
          - Bulk purchase savings
          - Store brand alternatives
          
          Present comparison in clear table format.
    toolsets:
      - product-catalog
  
  recipe_specialist:
    type: llm
    name: Recipe Specialist
    description: Finds and suggests recipes
    instructions:
      - if: "settings.dietary_restrictions != \"none\""
        content: |
          Only suggest recipes for: {settings.dietary_restrictions}
          Verify all ingredients comply with this restriction.

      - |
        Help users find recipes based on:
          - Available ingredients
          - Cuisine type
          - Meal type (breakfast, lunch, dinner, snack)
          - Cooking time available
          - Skill level
          
          Provide:
          - Recipe name and description
          - Ingredient list
          - Step-by-step instructions
          - Prep and cook time
          - Servings
          - Tips and variations
    toolsets:
      - duckduckgo
      - memory-server
```

**Use cases:**
- Weekly grocery shopping
- Meal planning
- Recipe discovery
- Price comparison
- Dietary restriction management

---

## Content Creation Teams

### Blog Writing Team

Comprehensive content creation with research and editing.

```yaml
version: "1"
id: blog-team
name: Blog Writing Team
description: Researches topics, writes SEO-optimized blog posts, and edits content for publication
interactive: true
default_agent: project_coordinator

settings:
  - name: seo_mode
    type: bool
    title: SEO Optimization
    description: Optimize content for search engines
    defaultValue: true
  
  - name: tone
    type: string
    title: Content Tone
    description: Writing style and formality
    defaultValue: conversational
  
  - name: target_length
    type: number
    title: Target Word Count
    description: Desired article length
    defaultValue: 1500

agents:
  project_coordinator:
    type: llm
    name: Project Coordinator
    description: Manages content creation workflow
    model: gpt-4o
    instructions:
      - |
        You coordinate the blog content creation process.
          
          Workflow:
          1. Clarify topic and requirements with user
          2. transfer_task to researcher for topic research
          3. transfer_task to content_creator (sequence agent handles draft + edit)
          4. Review final content with user
          5. Offer revisions if needed
          
          Keep user informed of progress at each stage.
    sub_agents:
      - researcher
      - content_creator
  
  researcher:
    type: llm
    name: Content Researcher
    description: Researches topics thoroughly
    model: anthropic
    instructions:
      - |
        Research the topic comprehensively using web search.
          
          Gather:
          - Key facts and statistics (with sources)
          - Current trends and developments
          - Expert opinions and quotes
          - Different perspectives on the topic
          - Related subtopics to cover
          
          Provide an organized research brief with:
          - Main points to cover
          - Supporting evidence
          - Suggested article structure
          - Source citations
    toolsets:
      - duckduckgo
      - memory-server
  
  content_creator:
    type: sequence
    name: Content Creation Pipeline
    description: Writes and edits content sequentially
    instructions:
      - |
        Create publication-ready content through:
          1. Draft writer creates initial version
          2. Editor polishes and refines
    sub_agents:
      - draft_writer
      - editor
  
  draft_writer:
    type: llm
    name: Draft Writer
    description: Creates initial content draft
    model: anthropic
    instructions:
      - if: "settings.tone == 'professional'"
        content: |
          Write in a professional tone:
          - Formal, authoritative language
          - Industry terminology
          - Third-person perspective
          - Data-driven arguments
      
      - if: "settings.tone == 'conversational'"
        content: |
          Write in a conversational tone:
          - Friendly, approachable language
          - Second-person perspective ("you")
          - Relatable examples and stories
          - Engaging, accessible style
      
      - if: "settings.tone == 'academic'"
        content: |
          Write in an academic tone:
          - Scholarly, formal language
          - Proper citations and references
          - Objective, analytical approach
          - Evidence-based reasoning
      
      - if: "settings.seo_mode"
        content: |
          Apply SEO best practices:
          - Identify target keyword from topic
          - Use keyword in title, headers, and throughout (1-2% density)
          - Create compelling meta description (under 160 chars)
          - Use headers hierarchically (H2, H3)
          - Include internal linking opportunities
          - Write descriptive alt text for images
      
      - |
        Write comprehensive blog post based on research.
          
          Target length: {settings.target_length} words
          
          Structure:
          1. Compelling introduction with hook
          2. Well-organized body with subheadings
          3. Practical examples and evidence
          4. Actionable takeaways
          5. Strong conclusion
          
          Use:
          - Clear, concise language
          - Short paragraphs (3-4 sentences)
          - Bullet points for lists
          - Relevant examples
    toolsets:
      - memory-server
  
  editor:
    type: llm
    name: Content Editor
    description: Reviews and polishes content
    model: anthropic
    instructions:
      - |
        Provide final editorial polish:
          
          Check for:
          - Grammar and spelling errors
          - Clarity and readability
          - Flow and transitions
          - Consistency in tone and style
          - Weak or unsupported claims
          - Redundancy or wordiness
          
          Improve:
          - Sentence structure variety
          - Word choice and precision
          - Paragraph transitions
          - Overall impact
          
          Maintain the author's voice while elevating quality.
          Verify word count is close to target: {settings.target_length}
```

**Use cases:**
- Blog post creation
- Content marketing
- Article writing
- SEO content
- Thought leadership

---

### Social Media Manager

Creates and optimizes social media content.

```yaml
version: "1"
id: social-media-team
name: Social Media Team
description: Creates engaging social media content optimized for different platforms
default_agent: content_strategist

settings:
  - name: platform
    type: string
    title: Platform
    description: Target social media platform
    defaultValue: twitter
  
  - name: brand_voice
    type: string
    title: Brand Voice
    description: Brand personality and tone
    defaultValue: friendly

agents:
  content_strategist:
    type: llm
    name: Content Strategist
    description: Creates platform-optimized social media content
    model: gpt-4o
    instructions:
      - if: "settings.platform == 'twitter'"
        content: |
          Create content for Twitter/X:
          - Maximum 280 characters
          - Use 1-2 relevant hashtags
          - Consider thread format for longer content
          - Engaging hook in first sentence
          - Include call-to-action when appropriate
      
      - if: "settings.platform == 'linkedin'"
        content: |
          Create content for LinkedIn:
          - Professional tone
          - Longer-form content (1300-1500 chars optimal)
          - Industry insights and thought leadership
          - Use line breaks for readability
          - 3-5 relevant hashtags
          - End with question to drive engagement
      
      - if: "settings.platform == 'facebook'"
        content: |
          Create content for Facebook:
          - Conversational, community-focused
          - Optimal length: 40-80 characters (or longer storytelling)
          - Encourage comments and shares
          - Use emojis sparingly
          - Include relevant link or call-to-action
      
      - if: "settings.platform == 'instagram'"
        content: |
          Create content for Instagram:
          - Visual-first approach (note image requirements)
          - Caption: engaging hook in first line
          - Use line breaks and emojis
          - 10-15 relevant hashtags (mix of popular and niche)
          - Clear call-to-action
      
      - if: "settings.brand_voice == 'professional'"
        content: |
          Brand voice: Professional
          - Authoritative and credible
          - Industry terminology
          - Data and insights
          - Polished and refined
      
      - if: "settings.brand_voice == 'friendly'"
        content: |
          Brand voice: Friendly
          - Warm and approachable
          - Conversational language
          - Personal touches
          - Relatable and authentic
      
      - if: "settings.brand_voice == 'humorous'"
        content: |
          Brand voice: Humorous
          - Witty and entertaining
          - Light-hearted tone
          - Clever wordplay
          - Fun and engaging
      
      - if: "settings.brand_voice == 'inspirational'"
        content: |
          Brand voice: Inspirational
          - Motivating and uplifting
          - Aspirational messaging
          - Storytelling approach
          - Emotional connection
      
      - |
        Create engaging social media content that:
          - Captures attention immediately
          - Provides value (educate, entertain, or inspire)
          - Encourages engagement
          - Aligns with brand voice
          - Follows platform best practices
          
          Offer multiple variations when helpful.
```

**Use cases:**
- Social media post creation
- Content calendar filling
- Platform optimization
- Brand voice consistency
- Engagement optimization

---

## Customer Support Teams

### Customer Support Team

Multi-channel support with specialized agents.

```yaml
version: "1"
id: support-team
name: Customer Support Team
description: Routes customer inquiries to specialized agents for technical issues, billing questions, and account management
interactive: true
default_agent: support_coordinator

settings:
  - name: priority_mode
    type: bool
    title: Priority Mode
    description: Fast-track urgent requests
    defaultValue: false

agents:
  support_coordinator:
    type: llm
    name: Support Coordinator
    description: Routes tickets to appropriate specialists
    instructions:
      - if: "settings.priority_mode"
        content: |
          PRIORITY MODE ACTIVE
          - Respond immediately
          - Escalate complex issues quickly
          - Set expectation for follow-up within 1 hour
          - Document all actions
      
      - |
        Greet the customer warmly and analyze their inquiry.
          
          Route as follows:
          - Bugs, errors, technical problems → transfer_task to technical_support
          - Payments, invoices, refunds, billing → transfer_task to billing_support
          - Login, password, settings, security → transfer_task to account_support
          
          If unclear, ask 1-2 clarifying questions before routing.
          Always summarize the issue when transferring.
    sub_agents:
      - technical_support
      - billing_support
      - account_support
    toolsets:
      - memory-server
  
  technical_support:
    type: llm
    name: Technical Support Specialist
    description: Handles technical issues and bugs
    model: gpt-4o
    instructions:
      - if: "settings.priority_mode"
        content: |
          Priority technical support:
          - Immediate acknowledgment
          - Rapid diagnosis
          - Escalate to engineering if complex
          - Commit to timeline
      
      - |
        Provide technical assistance:
          
          1. Gather information:
             - What were you trying to do?
             - What actually happened?
             - Error messages or screenshots?
             - When did this start?
          
          2. Diagnose systematically:
             - Check common issues first
             - Review error logs
             - Test reproduction steps
          
          3. Provide solutions:
             - Step-by-step instructions
             - Screenshots or videos when helpful
             - Link to relevant documentation
             - Verify resolution
          
          4. Escalate when needed:
             - Complex bugs
             - System outages
             - Security issues
          
          Be patient, clear, and thorough.
    toolsets:
      - memory-server
      - notification-server
  
  billing_support:
    type: llm
    name: Billing Support Specialist
    description: Handles billing and payment inquiries
    instructions:
      - |
        Address billing questions professionally:
          
          Common tasks:
          - Explain charges and invoices
          - Process refund requests (verify eligibility)
          - Update payment methods
          - Adjust subscriptions
          - Resolve billing disputes
          
          Guidelines:
          - Verify account ownership first
          - Be transparent about policies
          - Offer solutions when possible
          - Document all transactions
          - Confirm resolution
          
          Escalate to manager:
          - Refunds over $100
          - Policy exceptions
          - Unresolved disputes
    toolsets:
      - memory-server
  
  account_support:
    type: llm
    name: Account Support Specialist
    description: Handles account and security issues
    instructions:
      - |
        Assist with account management:
          
          Security-first approach:
          - Always verify identity before making changes
          - Use security questions
          - Send verification emails
          - Document all account changes
          
          Common tasks:
          - Password resets
          - Update profile information
          - Configure security settings (2FA)
          - Privacy and data requests
          - Account deletion or deactivation
          
          For account deletion:
          - Verify this is truly desired
          - Explain consequences (data loss)
          - Offer alternatives (deactivation)
          - Require explicit confirmation
    toolsets:
      - memory-server
      - notification-server
```

**Use cases:**
- Customer support routing
- Technical troubleshooting
- Billing assistance
- Account management
- Multi-tier support

---

## Business Process Teams

### Report Generator

Automated business report creation.

```yaml
version: "1"
id: report-team
name: Report Generator
description: Generates comprehensive business reports with data analysis and visualizations
default_agent: report_pipeline

settings:
  - name: report_type
    type: string
    title: Report Type
    description: Type of business report to generate
    defaultValue: sales

  - name: include_recommendations
    type: bool
    title: Include Recommendations
    description: Add strategic recommendations section
    defaultValue: true

agents:
  report_pipeline:
    type: sequence
    name: Report Generation Pipeline
    description: Creates reports through structured workflow
    instructions:
      - |
        Generate comprehensive business report through:
          1. Data collection and validation
          2. Statistical analysis
          3. Visualization creation
          4. Executive summary writing
    sub_agents:
      - data_collector
      - data_analyst
      - visualization_specialist
      - summary_writer
  
  data_collector:
    type: llm
    name: Data Collector
    description: Collects and validates source data
    instructions:
      - if: "settings.report_type == 'sales'"
        content: |
          Collect sales data:
          - Revenue by product/service
          - Sales by region
          - Customer acquisition/retention
          - Sales team performance
          - Period-over-period comparisons
      
      - if: "settings.report_type == 'marketing'"
        content: |
          Collect marketing data:
          - Campaign performance
          - Lead generation metrics
          - Conversion rates
          - ROI by channel
          - Brand awareness metrics
      
      - if: "settings.report_type == 'operations'"
        content: |
          Collect operations data:
          - Efficiency metrics
          - Resource utilization
          - Process cycle times
          - Quality metrics
          - Bottlenecks and delays
      
      - if: "settings.report_type == 'financial'"
        content: |
          Collect financial data:
          - Revenue and expenses
          - Profit margins
          - Cash flow
          - Budget vs. actual
          - Financial ratios
      
      - |
        Validate all data:
          - Check for missing values
          - Identify outliers
          - Verify data consistency
          - Note any data quality issues
          
          Organize data for analysis.
  
  data_analyst:
    type: llm
    name: Data Analyst
    description: Performs statistical analysis
    model: gpt-4o
    instructions:
      - |
        Analyze the collected data:
          
          Calculate:
          - Descriptive statistics (mean, median, etc.)
          - Trends over time
          - Period-over-period changes
          - Correlations and patterns
          
          Identify:
          - Key performance indicators
          - Significant trends
          - Anomalies or outliers
          - Areas of concern
          - Opportunities for improvement
          
          Provide clear, actionable insights.
  
  visualization_specialist:
    type: llm
    name: Visualization Specialist
    description: Creates data visualizations
    instructions:
      - |
          Recommend appropriate visualizations:
          
          - Trends over time → Line charts
          - Comparisons → Bar charts
          - Proportions → Pie charts
          - Relationships → Scatter plots
          - Distributions → Histograms
          
          For each visualization specify:
          - Chart type
          - Data to include
          - Axis labels
          - Title
          - Key insights to highlight
  
  summary_writer:
    type: llm
    name: Executive Summary Writer
    description: Writes executive summary
    model: anthropic
    instructions:
      - if: "settings.include_recommendations"
        content: |
          Create comprehensive executive summary with recommendations:
          
          Structure:
          1. Overview (2-3 sentences)
          2. Key Findings (bullet points, top 5)
          3. Detailed Analysis (paragraphs)
          4. Strategic Recommendations (numbered list)
          5. Next Steps (actionable items)
      
      - |
          Create executive summary:
          
          Structure:
          1. Overview (2-3 sentences)
          2. Key Findings (bullet points, top 5)
          3. Detailed Analysis (paragraphs)
          4. Conclusion
          
          Write for executive audience:
          - Clear and concise
          - Focus on business impact
          - Highlight actionable insights
          - Use business language
```

**Use cases:**
- Automated reporting
- Business analytics
- Performance tracking
- Executive dashboards
- Data-driven decision support

---

## Advanced Teams

### Multi-Perspective Analyzer

Parallel analysis from different viewpoints.

```yaml
version: "1"
id: analysis-team
name: Multi-Perspective Analysis Team
description: Analyzes proposals, decisions, or content from technical, business, and user perspectives simultaneously
default_agent: comprehensive_analyzer

agents:
  comprehensive_analyzer:
    type: parallel
    name: Comprehensive Analyzer
    description: Coordinates multi-perspective analysis
    instructions:
      - |
          Synthesize insights from all perspectives into a balanced analysis.
          
          Structure:
          1. Executive Summary
          2. Perspective Summaries:
             - Technical Assessment
             - Business Assessment
             - User Assessment
          3. Cross-Perspective Insights:
             - Agreements (all perspectives align)
             - Trade-offs (perspectives conflict)
             - Risks (concerns raised)
             - Opportunities (positive signals)
          4. Overall Recommendation
          
          Present a clear, actionable conclusion.
    sub_agents:
      - technical_analyst
      - business_analyst
      - user_analyst
  
  technical_analyst:
    type: llm
    name: Technical Analyst
    description: Evaluates technical feasibility and implications
    model: anthropic
    instructions:
      - |
          Analyze from a technical perspective:
          
          Assess:
          - Technical feasibility
          - Implementation complexity
          - Architecture implications
          - Scalability concerns
          - Performance impact
          - Security considerations
          - Technical debt
          - Maintenance burden
          
          Provide:
          - Technical risks (high/medium/low)
          - Implementation effort estimate
          - Technical dependencies
          - Alternative approaches
  
  business_analyst:
    type: llm
    name: Business Analyst
    description: Evaluates business value and strategy
    model: gpt-4o
    instructions:
      - |
          Analyze from a business perspective:
          
          Assess:
          - Business value and ROI
          - Market opportunity
          - Competitive advantage
          - Revenue impact
          - Cost implications
          - Resource requirements
          - Strategic alignment
          - Risk/reward profile
          
          Provide:
          - Business case strength
          - Expected timeline to value
          - Resource needs
          - Success metrics
  
  user_analyst:
    type: llm
    name: User Experience Analyst
    description: Evaluates user impact and usability
    model: anthropic
    instructions:
      - |
          Analyze from a user perspective:
          
          Assess:
          - User value and benefits
          - Usability and ease of use
          - User experience impact
          - Accessibility
          - Learning curve
          - Adoption barriers
          - User satisfaction potential
          - User risks or frustrations
          
          Provide:
          - User value assessment
          - UX risks and concerns
          - Adoption likelihood
          - User feedback needs
```

**Use cases:**
- Product proposals
- Feature decisions
- Architecture reviews
- Strategic planning
- Content evaluation

---

## Customization Tips

All examples can be customized:

### Adjust Instructions

Modify agent instructions to match your domain:
```yaml
instructions:
  - |
      [Your specific business rules]
      [Your domain knowledge]
      [Your quality standards]
```

### Add Settings

Extend with your own configuration:
```yaml
settings:
  - name: your_custom_setting
    type: string
    title: Your Setting
    description: What it controls
    allowed_values:
      - option1
      - option2
    defaultValue: option1
```

### Change Tools

Swap toolsets based on your needs:
```yaml
toolsets:
  - your-custom-tool
  - your-data-source
```

### Adjust Agents

Add, remove, or modify agents:
```yaml
agents:
  your_new_agent:
    # Configuration
```

## Next Steps

- **[Creating Teams](creating-agents.md)** - Customize these templates
- **[Configuration](configuration.md)** - Complete reference
- **[Agent Types](agent-types.md)** - Understanding execution patterns
- **[Tools](../tools/overview.md)** - Available integrations
- **[Advanced Patterns](../advanced/multi-agent.md)** - Complex workflows

---

**Related:**  
[Overview](overview.md) | [Creating Teams](creating-agents.md) | [Configuration](configuration.md)
