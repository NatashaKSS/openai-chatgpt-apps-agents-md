# AGENTS.md for OpenAI Apps SDK Development

## Project Overview

This document provides coding agents with comprehensive guidance for developing OpenAI Apps using the Apps SDK. The Apps SDK enables developers to create interactive components that run inside ChatGPT conversations, powered by Model Context Protocol (MCP) servers.

Apps SDK applications consist of:

- **MCP Server**: Backend that exposes tools ChatGPT can call, handles authentication, and returns structured data
- **UI Components**: React-based widgets that render inside ChatGPT using the `window.openai` API
- **Resource Templates**: HTML templates that define how components are displayed

## Development Environment Setup

### Prerequisites

- **Node.js 18+** or **Python 3.10+** (depending on your server choice)
- **npm/pnpm** for dependency management
- **TypeScript** knowledge recommended but not required
- **React 18+** for UI components
- **Git** for version control

### Core Dependencies

**For TypeScript/Node.js servers:**

```bash
npm install @modelcontextprotocol/sdk express cors
npm install -D typescript esbuild @types/node
```

**For Python servers:**

```bash
pip install fastapi uvicorn
# For FastMCP (recommended Python framework):
pip install fastmcp
```

**For UI components:**

```bash
npm install react@^18 react-dom@^18
npm install -D typescript esbuild
```

### Project Structure

Recommended project layout:

```
app/
├── server/                 # MCP server code
│   ├── main.py            # Python server entry
│   └── handlers/          # Tool handlers
├── web/                   # UI component source
│   ├── src/
│   │   ├── component.tsx  # Main React component
│   │   └── hooks/         # Custom hooks
│   ├── dist/              # Build output
│   ├── package.json
│   └── tsconfig.json
├── assets/                # Generated bundles
└── README.md
```

## Build Commands

### Component Build Process

1. **Create build script in `web/package.json`:**

```json
{
  "scripts": {
    "build": "esbuild src/component.tsx --bundle --format=esm --outfile=dist/component.js",
    "dev": "esbuild src/component.tsx --bundle --format=esm --outfile=dist/component.js --watch"
  }
}
```

2. **Build the component:**

```bash
cd web && npm run build
```

3. **For multiple components (using Vite):**

```bash
# Install Vite build orchestrator
npm install -D vite
# Run build-all script (if using OpenAI examples structure)
npm run build
```

### Server Development

**TypeScript MCP Server:**

```bash
# Install and start
npm install
npm start
# or
node server.js
```

**Python MCP Server:**

```bash
# FastAPI approach
uvicorn main:app --port 8000
# FastMCP approach
python server.py
```

## MCP Server Implementation

### Basic Server Structure (TypeScript)

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { readFileSync } from "node:fs";

const server = new McpServer({
  name: "my-app-server",
  version: "1.0.0",
});

// Load built component assets
const COMPONENT_JS = readFileSync("web/dist/component.js", "utf8");
const COMPONENT_CSS = readFileSync("web/dist/component.css", "utf8").catch(
  () => ""
);

// Register HTML resource template
server.registerResource(
  "my-widget",
  "ui://widget/my-component.html",
  {},
  async () => ({
    contents: [
      {
        uri: "ui://widget/my-component.html",
        mimeType: "text/html+skybridge",
        text: `
        <div id="root"></div>
        ${COMPONENT_CSS ? `<style>${COMPONENT_CSS}</style>` : ""}
        <script type="module">${COMPONENT_JS}</script>
      `.trim(),
        _meta: {
          "openai/widgetPrefersBorder": true,
          "openai/widgetCSP": {
            connect_domains: ["https://api.example.com"],
            resource_domains: ["https://cdn.example.com"],
          },
        },
      },
    ],
  })
);

// Register tool that uses the component
server.registerTool(
  "my-tool",
  {
    title: "My Tool",
    description: "Does something useful",
    inputSchema: {
      /* JSON schema */
    },
    _meta: {
      "openai/outputTemplate": "ui://widget/my-component.html",
      "openai/toolInvocation/invoking": "Processing...",
      "openai/toolInvocation/invoked": "Complete!",
    },
  },
  async (args) => {
    return {
      content: [{ type: "text", text: "Tool executed successfully" }],
      structuredContent: {
        /* Data for your component */
      },
      _meta: {
        /* Additional data hidden from model */
      },
    };
  }
);
```

### Basic Server Structure (Python with FastMCP)

```python
from fastmcp import FastMCP

mcp = FastMCP("My App Server")

@mcp.tool
def my_tool(query: str) -> dict:
    """Does something useful with the query."""
    return {
        "structuredContent": {"result": f"Processed: {query}"},
        "content": [{"type": "text", "text": "Tool executed"}]
    }

if __name__ == "__main__":
    mcp.run()
```

## UI Component Development

### Basic React Component Structure

```typescript
import React, { useEffect, useState } from "react";
import ReactDOM from "react-dom/client";

// Custom hook for accessing OpenAI globals
function useOpenAiGlobal<K extends keyof OpenAiGlobals>(key: K) {
  const [value, setValue] = useState(() => window.openai?.[key]);

  useEffect(() => {
    const handleGlobalsChange = (event: CustomEvent) => {
      const newValue = event.detail.globals[key];
      if (newValue !== undefined) {
        setValue(newValue);
      }
    };

    window.addEventListener("openai:set_globals", handleGlobalsChange);
    return () =>
      window.removeEventListener("openai:set_globals", handleGlobalsChange);
  }, [key]);

  return value;
}

// Main component
function MyComponent() {
  const toolOutput = useOpenAiGlobal("toolOutput");
  const theme = useOpenAiGlobal("theme");
  const [widgetState, setWidgetState] = useState(null);

  // Handle tool calls from component
  const handleAction = async () => {
    try {
      await window.openai?.callTool("refresh-data", { id: 123 });
    } catch (error) {
      console.error("Tool call failed:", error);
    }
  };

  // Persist widget state
  const updateState = (newState: any) => {
    setWidgetState(newState);
    window.openai?.setWidgetState(newState);
  };

  return (
    <div className={`app ${theme}`}>
      <h1>My Component</h1>
      {toolOutput?.structuredContent && (
        <div>Data: {JSON.stringify(toolOutput.structuredContent)}</div>
      )}
      <button onClick={handleAction}>Refresh</button>
    </div>
  );
}

// Mount component
const root = ReactDOM.createRoot(document.getElementById("root")!);
root.render(<MyComponent />);
```

## Testing Instructions

### Local Development Workflow

1. **Build your component:**

```bash
cd web && npm run build
```

2. **Start your MCP server:**

```bash
# TypeScript
npm start
# Python
python server.py
```

3. **Test with MCP Inspector:**

```bash
npx @modelcontextprotocol/inspector@latest
# Point to http://localhost:8000/mcp
```

4. **Expose via tunnel for ChatGPT testing:**

```bash
ngrok http 8000
# Use the HTTPS URL in ChatGPT developer mode
```

### Testing Checklist

- [ ] MCP server starts without errors
- [ ] All tools list correctly in Inspector
- [ ] Tool calls return expected structured content
- [ ] Components render without console errors
- [ ] Widget state persists across interactions
- [ ] Authentication flows work (if implemented)
- [ ] Mobile layouts render correctly
- [ ] External API calls succeed (if used)

### Common Testing Tools

- **MCP Inspector**: `npx @modelcontextprotocol/inspector@latest`
- **MCPJam Inspector**: `npx @mcpjam/inspector@latest` (supports Apps SDK UI testing)
- **ngrok**: For exposing local servers to ChatGPT
- **Browser DevTools**: For component debugging

## Error Handling Best Practices

### Server-Side Error Handling

```typescript
server.registerTool("my-tool", {}, async (args) => {
  try {
    const result = await someApiCall(args);
    return {
      content: [{ type: "text", text: "Success" }],
      structuredContent: result,
    };
  } catch (error) {
    console.error("Tool error:", error);
    return {
      content: [
        {
          type: "text",
          text: `Error: ${error.message}`,
        },
      ],
      structuredContent: { error: true, message: error.message },
    };
  }
});
```

### Client-Side Error Handling

```typescript
// Wrap tool calls in try-catch
const callTool = async (name: string, args: any) => {
  try {
    return await window.openai?.callTool(name, args);
  } catch (error) {
    console.error(`Tool ${name} failed:`, error);
    setError(`Failed to ${name}: ${error.message}`);
    return null;
  }
};
```

## Authentication Implementation

### OAuth 2.1 Setup (if required)

```typescript
import { requireBearerAuth } from "@modelcontextprotocol/sdk/server/auth/middleware/bearerAuth.js";

// Configure OAuth middleware
const authMiddleware = requireBearerAuth({
  verifier: tokenVerifier,
  requiredScopes: ["mcp:tools"],
  resourceMetadataUrl: serverUrl,
});

// Apply to MCP routes
app.use("/mcp", authMiddleware);
```

## Deployment Configuration

### Environment Variables

```bash
# Server configuration
HOST=localhost
PORT=8000
NODE_ENV=production

# External APIs
API_KEY=your-api-key
DATABASE_URL=your-db-url

# OAuth (if using)
OAUTH_CLIENT_ID=your-client-id
OAUTH_CLIENT_SECRET=your-client-secret
```

### Production Deployment

**Docker (recommended):**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 8000
CMD ["npm", "start"]
```

**Deployment platforms:**

- **Fly.io, Render, Railway**: Managed containers with auto TLS
- **Google Cloud Run, Azure Container Apps**: Serverless containers
- **Heroku**: Platform-as-a-service (legacy)

### Security Considerations

- Always use HTTPS in production
- Implement proper CORS policies
- Validate all user inputs
- Store secrets in environment variables
- Use Content Security Policy for components
- Rate limit API endpoints
- Implement proper error logging

## Advanced Features

### Component-Initiated Tool Calls

Mark tools as widget-accessible:

```typescript
_meta: {
  "openai/outputTemplate": "ui://widget/my-component.html",
  "openai/widgetAccessible": true
}
```

### Display Mode Management

```typescript
// Request fullscreen mode
await window.openai?.requestDisplayMode({ mode: "fullscreen" });

// Handle display mode changes
const displayMode = useOpenAiGlobal("displayMode");
```

### Localization Support

```typescript
// Server-side locale detection
const locale = request._meta?.["openai/locale"] || "en";

// Component-side locale usage
const locale = useOpenAiGlobal("locale");
```

## Common Pitfalls to Avoid

1. **Don't** combine tool calls with text output in responses
2. **Don't** expose sensitive data in `structuredContent` (use `_meta` instead)
3. **Don't** forget to handle async operations properly
4. **Don't** skip component error boundaries
5. **Don't** make components too heavy (impacts ChatGPT performance)
6. **Don't** forget to test mobile layouts
7. **Don't** hardcode URLs or API keys
8. **Don't** skip input validation on tool parameters

## Performance Optimization

- Keep component bundles under 1MB
- Minimize external dependencies
- Use lazy loading for heavy components
- Implement proper caching strategies
- Optimize images and assets
- Use compression for API responses
- Monitor bundle sizes with build tools

## Resources and References

- **Official Apps SDK Docs**: https://developers.openai.com/apps-sdk/
- **MCP Specification**: https://modelcontextprotocol.io/
- **Example Repository**: https://github.com/openai/openai-apps-sdk-examples
- **TypeScript SDK**: https://github.com/modelcontextprotocol/typescript-sdk
- **Python SDK**: https://github.com/modelcontextprotocol/python-sdk
- **FastMCP Framework**: https://github.com/jlowin/fastmcp
