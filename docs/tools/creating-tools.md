# Creating Custom MCP Tools

Build custom MCP servers to extend agent capabilities.

## Overview

While MyAgents provides several built-in tools, you can create custom MCP servers to integrate your own services, APIs, and data sources.

## MCP Server Types

### In-Process Servers

TypeScript/JavaScript servers that run in the same process as the API.

**Advantages:**
- No network overhead
- Direct access to system resources
- Easy debugging
- Fast execution

**Use for:**
- Internal business logic
- Database operations
- File system access
- System integrations

### HTTP/SSE Servers

External servers accessed via HTTP or Server-Sent Events.

**Advantages:**
- Language agnostic
- Can run on separate infrastructure
- Scalable independently
- Isolated failures

**Use for:**
- External APIs
- Third-party services
- Microservices
- Cross-platform tools

### STDIO Servers

Servers that communicate via standard input/output.

**Advantages:**
- Simple protocol
- Works with any language
- Good for command-line tools
- Lightweight

**Use for:**
- CLI tool wrappers
- Legacy system integration
- Simple scripts

---

## Creating an In-Process Server

### Basic Structure

```typescript
// src/tools/mcp/servers/CustomTool.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { 
  ListToolsRequestSchema, 
  CallToolRequestSchema 
} from '@modelcontextprotocol/sdk/types.js';

export class CustomToolServer {
  private server: Server;

  constructor() {
    this.server = new Server({
      name: 'custom-tool',
      version: '1.0.0'
    }, {
      capabilities: {
        tools: {}
      }
    });

    this.setupHandlers();
  }

  private setupHandlers(): void {
    // List available tools
    this.server.setRequestHandler(
      ListToolsRequestSchema,
      async () => ({
        tools: [
          {
            name: 'do_something',
            description: 'Does something useful',
            inputSchema: {
              type: 'object',
              properties: {
                input: {
                  type: 'string',
                  description: 'Input parameter'
                }
              },
              required: ['input']
            }
          }
        ]
      })
    );

    // Handle tool calls
    this.server.setRequestHandler(
      CallToolRequestSchema,
      async (request) => {
        const { name, arguments: args } = request.params;

        if (name === 'do_something') {
          return await this.doSomething(args.input as string);
        }

        throw new Error(`Unknown tool: ${name}`);
      }
    );
  }

  private async doSomething(input: string): Promise<any> {
    // Your tool logic here
    return {
      content: [
        {
          type: 'text',
          text: `Processed: ${input}`
        }
      ]
    };
  }

  public getServer(): Server {
    return this.server;
  }
}

// Export for registration
export const CustomTool = CustomToolServer;
```

### Register the Server

Add to `mcp.servers.json`:

```json
{
  "custom-tool": {
    "type": "inprocess",
    "module": "@/tools/mcp/servers/CustomTool",
    "exportName": "CustomToolServer"
  }
}
```

---

## Creating an HTTP Server

### Server Implementation

```python
# external-tool-server.py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/mcp', methods=['POST'])
def handle_mcp():
    data = request.json
    method = data.get('method')
    
    if method == 'tools/list':
        return jsonify({
            'tools': [
                {
                    'name': 'external_operation',
                    'description': 'Performs an external operation',
                    'inputSchema': {
                        'type': 'object',
                        'properties': {
                            'param': {
                                'type': 'string',
                                'description': 'Operation parameter'
                            }
                        },
                        'required': ['param']
                    }
                }
            ]
        })
    
    elif method == 'tools/call':
        tool_name = data['params']['name']
        arguments = data['params']['arguments']
        
        if tool_name == 'external_operation':
            result = perform_operation(arguments['param'])
            return jsonify({
                'content': [
                    {
                        'type': 'text',
                        'text': result
                    }
                ]
            })
    
    return jsonify({'error': 'Unknown method'}), 400

def perform_operation(param):
    # Your logic here
    return f"Operation completed with: {param}"

if __name__ == '__main__':
    app.run(port=3001)
```

### Register HTTP Server

Add to `mcp.servers.json`:

```json
{
  "external-tool": {
    "type": "http",
    "url": "http://localhost:3001/mcp"
  }
}
```

---

## Tool Input Schema

Define clear input schemas using JSON Schema:

```typescript
{
  name: 'search_products',
  description: 'Searches for products in the catalog',
  inputSchema: {
    type: 'object',
    properties: {
      query: {
        type: 'string',
        description: 'Search query'
      },
      category: {
        type: 'string',
        description: 'Product category filter',
        enum: ['electronics', 'clothing', 'food']
      },
      maxResults: {
        type: 'number',
        description: 'Maximum number of results',
        default: 10,
        minimum: 1,
        maximum: 100
      },
      priceRange: {
        type: 'object',
        properties: {
          min: { type: 'number' },
          max: { type: 'number' }
        }
      }
    },
    required: ['query']
  }
}
```

---

## Best Practices

### 1. Clear Tool Names

Use descriptive, action-oriented names:

**Good:**
- `search_products`
- `send_email`
- `create_calendar_event`

**Avoid:**
- `tool1`
- `handler`
- `process`

### 2. Comprehensive Descriptions

```typescript
{
  name: 'send_email',
  description: 'Sends an email message. Requires valid email addresses for sender and recipients. Supports HTML and plain text content. Returns delivery status.',
  inputSchema: {...}
}
```

### 3. Input Validation

Validate all inputs before processing:

```typescript
private async sendEmail(args: any): Promise<any> {
  // Validate
  if (!args.to || !args.subject || !args.body) {
    throw new Error('Missing required fields: to, subject, body');
  }
  
  if (!this.isValidEmail(args.to)) {
    throw new Error('Invalid email address');
  }
  
  // Process
  const result = await this.emailService.send(args);
  return result;
}
```

### 4. Error Handling

Return clear error messages:

```typescript
try {
  const result = await this.performOperation(args);
  return {
    content: [
      { type: 'text', text: JSON.stringify(result) }
    ]
  };
} catch (error) {
  return {
    content: [
      { 
        type: 'text', 
        text: `Error: ${error.message}` 
      }
    ],
    isError: true
  };
}
```

### 5. Logging

Log all tool operations:

```typescript
private async doSomething(input: string): Promise<any> {
  this.logger.info('Tool called', { tool: 'do_something', input });
  
  try {
    const result = await this.process(input);
    this.logger.info('Tool completed', { tool: 'do_something', success: true });
    return result;
  } catch (error) {
    this.logger.error('Tool failed', { tool: 'do_something', error });
    throw error;
  }
}
```

---

## Testing Custom Tools

### Unit Tests

```typescript
import { CustomToolServer } from '@/tools/mcp/servers/CustomTool';

describe('CustomToolServer', () => {
  let server: CustomToolServer;

  beforeEach(() => {
    server = new CustomToolServer();
  });

  it('should list available tools', async () => {
    const response = await server.getServer().request({
      method: 'tools/list',
      params: {}
    });
    
    expect(response.tools).toHaveLength(1);
    expect(response.tools[0].name).toBe('do_something');
  });

  it('should execute tool successfully', async () => {
    const response = await server.getServer().request({
      method: 'tools/call',
      params: {
        name: 'do_something',
        arguments: { input: 'test' }
      }
    });
    
    expect(response.content[0].text).toContain('Processed: test');
  });

  it('should handle errors gracefully', async () => {
    await expect(server.getServer().request({
      method: 'tools/call',
      params: {
        name: 'unknown_tool',
        arguments: {}
      }
    })).rejects.toThrow('Unknown tool');
  });
});
```

### Integration Tests

Test tools with actual agents:

```typescript
describe('CustomTool Integration', () => {
  it('should work with agent', async () => {
    const agent = await createTestAgent({
      toolsets: ['custom-tool']
    });
    
    const response = await agent.processMessage(
      'Use the custom tool with input "test"'
    );
    
    expect(response).toContain('Processed: test');
  });
});
```

---

## Security Considerations

### 1. Authentication

Implement authentication for sensitive operations:

```typescript
private validateAuth(token: string): boolean {
  // Verify JWT, API key, etc.
  return this.authService.validate(token);
}
```

### 2. Rate Limiting

Prevent abuse:

```typescript
private rateLimiter = new RateLimiter({
  maxRequests: 100,
  windowMs: 60000 // 1 minute
});

private async doSomething(input: string): Promise<any> {
  if (!this.rateLimiter.checkLimit()) {
    throw new Error('Rate limit exceeded');
  }
  // ... proceed
}
```

### 3. Input Sanitization

Sanitize all inputs:

```typescript
private sanitizeInput(input: string): string {
  return input
    .trim()
    .replace(/[<>]/g, '')  // Remove dangerous characters
    .substring(0, 1000);   // Limit length
}
```

### 4. Access Control

Check permissions:

```typescript
private async checkPermission(userId: string, operation: string): Promise<boolean> {
  return await this.permissionService.hasPermission(userId, operation);
}
```

---

## Deployment

### Development

```bash
# Start API with custom tool
npm run dev
```

### Production

```bash
# Build
npm run build

# Start production server
npm start
```

### Docker

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY dist ./dist
COPY src/tools/mcp/servers ./dist/tools/mcp/servers

CMD ["node", "dist/server.js"]
```

---

## Example: Weather Tool

Complete example of a custom weather tool:

```typescript
// src/tools/mcp/servers/WeatherTool.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import axios from 'axios';

export class WeatherToolServer {
  private server: Server;
  private apiKey: string;

  constructor(apiKey: string) {
    this.apiKey = apiKey;
    this.server = new Server({
      name: 'weather-tool',
      version: '1.0.0'
    }, {
      capabilities: { tools: {} }
    });

    this.setupHandlers();
  }

  private setupHandlers(): void {
    this.server.setRequestHandler(
      ListToolsRequestSchema,
      async () => ({
        tools: [
          {
            name: 'get_weather',
            description: 'Gets current weather for a location',
            inputSchema: {
              type: 'object',
              properties: {
                location: {
                  type: 'string',
                  description: 'City name or zip code'
                },
                units: {
                  type: 'string',
                  description: 'Temperature units',
                  enum: ['metric', 'imperial'],
                  default: 'metric'
                }
              },
              required: ['location']
            }
          }
        ]
      })
    );

    this.server.setRequestHandler(
      CallToolRequestSchema,
      async (request) => {
        const { name, arguments: args } = request.params;

        if (name === 'get_weather') {
          return await this.getWeather(
            args.location as string,
            args.units as string || 'metric'
          );
        }

        throw new Error(`Unknown tool: ${name}`);
      }
    );
  }

  private async getWeather(location: string, units: string): Promise<any> {
    try {
      const response = await axios.get(
        `https://api.weatherapi.com/v1/current.json`,
        {
          params: {
            key: this.apiKey,
            q: location,
            units
          }
        }
      );

      const data = response.data;
      const result = {
        location: data.location.name,
        temperature: data.current.temp_c,
        condition: data.current.condition.text,
        humidity: data.current.humidity,
        windSpeed: data.current.wind_kph
      };

      return {
        content: [
          {
            type: 'text',
            text: JSON.stringify(result, null, 2)
          }
        ]
      };
    } catch (error) {
      return {
        content: [
          {
            type: 'text',
            text: `Error fetching weather: ${error.message}`
          }
        ],
        isError: true
      };
    }
  }

  public getServer(): Server {
    return this.server;
  }
}

export const WeatherTool = WeatherToolServer;
```

Register in `mcp.servers.json`:

```json
{
  "weather-tool": {
    "type": "inprocess",
    "module": "@/tools/mcp/servers/WeatherTool",
    "exportName": "WeatherToolServer",
    "args": ["${env:WEATHER_API_KEY}"]
  }
}
```

---

## Next Steps

- **[Available Tools](available-tools.md)** - Study existing tools
- **[Using Tools](using-tools.md)** - Integration patterns
- **MCP Documentation** - Official protocol documentation

---

**Related:**  
[Tools Overview](overview.md) | [Available Tools](available-tools.md) | [Using Tools](using-tools.md)

