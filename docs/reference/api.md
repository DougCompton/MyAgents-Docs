# API Reference

REST API endpoints for the MyAgents platform.

## Base URL

```
https://api.myagents.example.com
```

Development:
```
http://localhost:3000
```

## Authentication

All API requests require authentication via Bearer token.

### Headers

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Getting Access Tokens

See [Authentication Endpoints](#authentication) below.

---

## Authentication

### Register

Create a new user account.

```http
POST /auth/register
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123",
  "firstName": "John",
  "lastName": "Doe"
}
```

**Response:** `201 Created`
```json
{
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "user": {
    "id": "user-id",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe"
  }
}
```

---

### Login

Authenticate and receive tokens.

```http
POST /auth/login
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "user": {
    "id": "user-id",
    "email": "user@example.com"
  }
}
```

---

### Refresh Token

Get new access token using refresh token.

```http
POST /auth/refresh
```

**Headers:**
```http
Authorization: Bearer <refresh_token>
```

**Request Body:**
```json
{
  "refreshToken": "eyJhbGc..."
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc..."
}
```

---

### Logout

Invalidate tokens.

```http
POST /auth/logout
```

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "refreshToken": "eyJhbGc..."
}
```

**Response:** `200 OK`
```json
{
  "message": "Logged out successfully"
}
```

---

## Chat

### Send Message

Send a message to an agent.

```http
POST /chat/messages
```

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "message": "Hello, I need help with something",
  "sessionId": "optional-session-id",
  "agentId": "optional-agent-id"
}
```

**Response:** `200 OK`
```json
{
  "id": "message-id",
  "sessionId": "session-id",
  "message": "Hello, I need help with something",
  "response": "Hello! I'm here to help. What do you need assistance with?",
  "agentId": "switchboard",
  "timestamp": "2025-10-25T10:30:00Z"
}
```

---

### Stream Message

Stream agent response in real-time.

```http
POST /chat/stream
```

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: text/event-stream
```

**Request Body:**
```json
{
  "message": "Write a story about AI",
  "sessionId": "optional-session-id"
}
```

**Response:** `200 OK`
```
Content-Type: text/event-stream

data: {"type":"start","sessionId":"session-123"}

data: {"type":"chunk","content":"Once upon"}

data: {"type":"chunk","content":" a time"}

data: {"type":"chunk","content":" there was"}

data: {"type":"end","messageId":"msg-123"}
```

---

### Get Chat History

Retrieve conversation history.

```http
GET /chat/history?sessionId={sessionId}&limit={limit}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sessionId | string | Yes | Session identifier |
| limit | number | No | Max messages (default: 50) |

**Response:** `200 OK`
```json
{
  "messages": [
    {
      "id": "msg-1",
      "role": "user",
      "content": "Hello",
      "timestamp": "2025-10-25T10:00:00Z"
    },
    {
      "id": "msg-2",
      "role": "assistant",
      "content": "Hi! How can I help?",
      "timestamp": "2025-10-25T10:00:01Z"
    }
  ],
  "sessionId": "session-123",
  "total": 2
}
```

---

## Agents

### List Agents

Get all available agents.

```http
GET /agents
```

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Response:** `200 OK`
```json
{
  "agents": [
    {
      "id": "switchboard",
      "name": "Switchboard",
      "description": "Routes requests to appropriate agents",
      "interactive": true
    },
    {
      "id": "blog-team",
      "name": "Blog Writing Team",
      "description": "Creates blog content through research and writing",
      "interactive": true
    }
  ]
}
```

---

### Get Agent Details

Get detailed information about a specific agent.

```http
GET /agents/{agentId}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| agentId | string | Yes | Agent identifier |

**Response:** `200 OK`
```json
{
  "id": "blog-team",
  "name": "Blog Writing Team",
  "description": "Creates blog content",
  "interactive": true,
  "settings": [
    {
      "name": "contentLength",
      "title": "Content Length",
      "type": "string",
      "description": "Target length (short, medium, long)",
      "defaultValue": "medium"
    }
  ],
  "personas": ["coordinator", "researcher", "writer", "editor"]
}
```

---

## Memory

### Create Memory

Store information for later retrieval.

```http
POST /memory
```

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "content": "User prefers organic products",
  "category": "preferences",
  "agentId": "shopping-assistant"
}
```

**Response:** `201 Created`
```json
{
  "id": "memory-id",
  "userId": "user-id",
  "agentId": "shopping-assistant",
  "content": "User prefers organic products",
  "category": "preferences",
  "createdAt": "2025-10-25T10:30:00Z"
}
```

---

### List Memories

Retrieve stored memories.

```http
GET /memory?category={category}&agentId={agentId}&limit={limit}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| category | string | No | Filter by category |
| agentId | string | No | Filter by agent |
| limit | number | No | Max results (default: 50) |

**Response:** `200 OK`
```json
{
  "memories": [
    {
      "id": "memory-1",
      "content": "User prefers organic products",
      "category": "preferences",
      "agentId": "shopping-assistant",
      "createdAt": "2025-10-25T10:00:00Z"
    }
  ],
  "total": 1
}
```

---

### Delete Memory

Remove a stored memory.

```http
DELETE /memory/{memoryId}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| memoryId | string | Yes | Memory identifier |

**Response:** `200 OK`
```json
{
  "message": "Memory deleted successfully"
}
```

---

## Health

### Health Check

Check API status.

```http
GET /health
```

**Response:** `200 OK`
```json
{
  "status": "ok",
  "timestamp": "2025-10-25T10:30:00Z",
  "services": {
    "database": "ok",
    "mcpRegistry": "ok"
  }
}
```

---

## Error Responses

### Error Format

All errors follow this format:

```json
{
  "message": "Error description",
  "error": "ErrorType",
  "statusCode": 400
}
```

### Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Missing or invalid auth |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 500 | Internal Server Error | Server error |

### Common Errors

**401 Unauthorized**
```json
{
  "message": "Authentication required",
  "error": "Unauthorized",
  "statusCode": 401
}
```

**400 Bad Request**
```json
{
  "message": "Invalid request body",
  "error": "ValidationError",
  "statusCode": 400
}
```

**404 Not Found**
```json
{
  "message": "Agent not found",
  "error": "NotFound",
  "statusCode": 404
}
```

---

## Rate Limiting

### Limits

- **Anonymous**: 100 requests/hour
- **Authenticated**: 1000 requests/hour
- **Premium**: 10,000 requests/hour

### Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1635264000
```

### Rate Limit Exceeded

**Response:** `429 Too Many Requests`
```json
{
  "message": "Rate limit exceeded",
  "error": "RateLimitExceeded",
  "statusCode": 429,
  "retryAfter": 3600
}
```

---

## SDK Usage

The MyAgents platform provides a TypeScript SDK for easier integration.

### Installation

```bash
npm install @myagents/api-client
```

### Usage

```typescript
import { getAuthenticatedApiClient, sendMessage } from '@myagents/api-client';

// Get authenticated client
const client = getAuthenticatedApiClient();

// Send message
const response = await sendMessage({
  client: client.getClient(),
  body: {
    message: "Hello, agent!",
    sessionId: "my-session"
  }
});

console.log(response.data.response);
```

### Benefits

- ✅ TypeScript type safety
- ✅ Automatic authentication
- ✅ Error handling
- ✅ Request/response validation

---

## Pagination

### Request Parameters

```http
GET /resource?page=1&limit=20
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | number | 1 | Page number |
| limit | number | 20 | Items per page |

### Response Format

```json
{
  "items": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5,
    "hasMore": true
  }
}
```

---

## Webhook Events

### Event Types

- `message.sent` - Message sent to agent
- `message.received` - Response received from agent
- `session.created` - New chat session
- `agent.switched` - Agent changed

### Webhook Format

```json
{
  "event": "message.received",
  "timestamp": "2025-10-25T10:30:00Z",
  "data": {
    "sessionId": "session-123",
    "messageId": "msg-456",
    "content": "Agent response..."
  }
}
```

---

## Next Steps

- **[JSON Schema](schema.md)** - Complete schema reference
- **[Creating Agents](../agents/creating-agents.md)** - Build custom agents
- **[Tools](../tools/available-tools.md)** - Explore available tools

---

**Related:**  
[Schema Reference](schema.md) | [Authentication Guide](../getting-started.md#authentication) | [SDK Documentation](https://docs.myagents.example.com/sdk)

