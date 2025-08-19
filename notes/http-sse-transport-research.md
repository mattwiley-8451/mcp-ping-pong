# Research Report: Adding HTTP and SSE Transports to MCP Server

## Executive Summary
This report outlines the requirements and implementation approach for adding HTTP and Server-Sent Events (SSE) transports to the existing MCP ping-pong server, while maintaining stdio transport compatibility.

## Current State
- Simple MCP server using `StdioServerTransport`
- Single `ping` tool responding with "pong"
- Communicates via stdin/stdout only

## Impact on Existing Stdio Transport
**Adding HTTP/SSE transports does NOT remove or prevent stdio transport usage.** The transports are independent and can coexist:
- You can run separate servers for each transport type
- You can create a unified server that supports multiple transports simultaneously
- The MCP SDK is designed to allow multiple transport methods
- Each transport handles different connection types (stdio for direct process communication, HTTP/SSE for web-based clients)

## Required Changes

### Transport Layer Updates
- **What**: Replace/supplement `StdioServerTransport` with `StreamableHTTPServerTransport`
- **How**: Import new transport class and configure with Express.js server
- **Why**: Modern unified approach supporting both HTTP POST and SSE streaming per current MCP spec

### HTTP Server Infrastructure
- **What**: Add Express.js server with `/mcp` endpoint
- **How**: Create server handling GET (SSE), POST (JSON-RPC), DELETE (session termination)
- **Why**: Required for HTTP-based client connections and web browser compatibility

### Session Management
- **What**: Implement session-based transport mapping
- **How**: Use session IDs, event stores, and lifecycle callbacks
- **Why**: Enable stateful connections, resumability, and proper resource cleanup

### Security & CORS Configuration
- **What**: Add cross-origin request handling and DNS rebinding protection
- **How**: Configure CORS middleware and transport security options
- **Why**: Essential for web-based clients and production security

## Implementation Approach Options

### Option A: Multi-Transport Server
- Single server supporting both stdio and HTTP/SSE
- Transport detection based on startup parameters
- Shared core MCP logic

### Option B: Separate Servers
- Keep existing stdio server unchanged
- Create new HTTP/SSE server
- Duplicate core logic or extract to shared module

### Option C: Unified Server (Recommended)
- Run both transports simultaneously
- Stdio for CLI/process integration
- HTTP/SSE for web clients

## Task List

1. **Update Dependencies**
   - Add `express` and `cors` to package.json
   - Install new dependencies

2. **Extract Core Logic**
   - Move tool definitions and handlers to shared module
   - Create reusable server configuration

3. **Implement HTTP Transport**
   - Create Express server with `/mcp` endpoint
   - Configure `StreamableHTTPServerTransport`
   - Set up session management and security

4. **Update Server Entry Points**
   - Modify existing server.js or create new http-server.js
   - Add transport selection logic

5. **Update Documentation**
   - Document HTTP/SSE usage in README.md
   - Include endpoint information and client examples

6. **Testing & Validation**
   - Test stdio transport (existing functionality)
   - Test HTTP POST requests
   - Test SSE streaming connections

## Dependencies

### New Dependencies
- `express` - HTTP server framework
- `cors` - Cross-origin request handling

### Existing Dependencies (unchanged)
- `@modelcontextprotocol/sdk` - Already includes SSE and HTTP utilities

### Development Dependencies (optional)
- Testing frameworks for HTTP endpoint testing
- Client libraries for validation

## Technical Implementation Details

### StreamableHTTPServerTransport Configuration
```javascript
const transport = new StreamableHTTPServerTransport({
  sessionIdGenerator: () => randomUUID(), // Enable stateful sessions
  enableJsonResponse: false, // Use SSE by default
  eventStore: new InMemoryEventStore(), // Enable resumability
  onsessioninitialized: (sessionId) => { /* callback */ },
  onsessionclosed: (sessionId) => { /* callback */ },
  allowedHosts: ['localhost'], // DNS rebinding protection
  allowedOrigins: ['*'], // CORS configuration
  enableDnsRebindingProtection: true
});
```

### Endpoint Structure
- **GET /mcp**: Establishes SSE stream for real-time communication
- **POST /mcp**: Sends JSON-RPC messages
- **DELETE /mcp**: Terminates session

## Benefits
- **Backward Compatibility**: Stdio transport remains functional
- **Web Integration**: Enables browser-based MCP clients
- **Flexibility**: Multiple connection methods for different use cases
- **Modern Protocol**: Follows current MCP specification (2025-06-18)
- **Session Management**: Built-in resumability and state management
- **Security**: DNS rebinding protection and CORS support

## Risks & Considerations
- **Complexity**: Increased codebase complexity with multiple transports
- **Security**: HTTP endpoints require proper security configuration
- **Resource Usage**: HTTP server consumes additional system resources
- **Port Management**: HTTP server requires available port
- **CORS Configuration**: Proper cross-origin setup needed for web clients

## Conclusion
This implementation maintains the simplicity of the current ping-pong server while extending its capabilities for web-based integration. The recommended approach is to create a unified server supporting both transports, allowing maximum flexibility for different client types while preserving existing functionality.