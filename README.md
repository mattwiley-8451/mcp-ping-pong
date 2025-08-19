# MCP Ping Pong Server

A simple Model Context Protocol (MCP) server that provides a `ping` tool which responds with "pong".

## Installation

```bash
npm install
```

## Running the Server

```bash
npm start
```

Or directly:

```bash
node server.js
```

## Tools

- **ping**: A simple tool that responds with "pong"

## Usage

The server communicates over stdio using the MCP protocol. It can be used with any MCP-compatible client.

### Example - Unpackaged Local Runner

This demonstrates how to configure the MCP ping-pong server by pulling down the git repo and running the server locally.

```json
{
  "mcpServers": {
    "ping-pong": {
      "command": "node",
      "args": [
        "/path/to/repo/mcp-ping-pong/server.js"
      ]
    }
  }
}
```