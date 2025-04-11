
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 3/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Overview

This lesson covers the fundamentals of MCP client development, including setup, connection, and state management.

## Setting up an MCP Client

An MCP client connects to MCP servers to access their capabilities. Setting up a client involves:

1. **Choosing a programming language and SDK**
   - Several official SDKs are available
   - TypeScript/JavaScript
   - Python
   - Java/Spring
   - C#/.NET
   - Kotlin
   - Swift

2. **Configuring transport mechanisms**
   - HTTP with Server-Sent Events (SSE)
   - WebSockets
   - STDIO
   - UNIX sockets

3. **Implementing connection management**
   - Connection establishment
   - Reconnection strategies
   - Connection pooling
   - Timeout handling

4. **Setting up authentication**
   - OAuth token management
   - API key handling
   - Credential storage
   - Token refresh

5. **Handling capability discovery**
   - Server capability retrieval
   - Capability caching
   - Capability filtering
   - Version negotiation

## Connection and Handshake Process

The connection process includes:

1. **Establishing transport connection**
   - Creating the appropriate connection based on transport
   - Setting connection parameters
   - Configuring timeouts
   - Setting up event handlers

2. **Protocol version negotiation**
   - Client sends supported versions
   - Server responds with selected version
   - Compatibility checking
   - Fallback strategies

3. **Authentication**
   - Sending authentication credentials
   - Token validation
   - Handling authentication failures
   - Refreshing expired tokens

4. **Capability discovery**
   - Requesting available capabilities
   - Processing capability metadata
   - Filtering capabilities based on permissions
   - Storing capabilities for later use

5. **Connection maintenance**
   - Heartbeat mechanism
   - Connection monitoring
   - Reconnection on failure
   - Graceful disconnection

## Discovering Server Capabilities

Capability discovery allows clients to:

- **Retrieve available tools, resources, and prompts**
  - Get comprehensive list of capabilities
  - Filter by type (tool, resource, prompt)
  - Filter by category or namespace
  - Search by name or description

- **Access capability metadata and descriptions**
  - Human-readable descriptions
  - Parameter schemas
  - Return value schemas
  - Usage examples

- **Understand input/output schemas**
  - Parameter types and constraints
  - Required vs. optional parameters
  - Default values
  - Return value structure

- **Retrieve usage examples**
  - Sample parameter values
  - Example responses
  - Error scenarios
  - Usage guidance

- **Check authentication requirements**
  - Required permissions
  - Scope requirements
  - Rate limits
  - Access restrictions

## Managing Client State

Client state management includes:

1. **Connection status tracking**
   - Connected
   - Connecting
   - Disconnected
   - Reconnecting
   - Error states

2. **Session management**
   - Session creation
   - Session persistence
   - Session expiration
   - Session recovery

3. **Capability caching**
   - Storing discovered capabilities
   - Cache invalidation
   - Refreshing capability data
   - Capability versioning

4. **Authentication token management**
   - Token storage
   - Token expiration
   - Token refresh
   - Token revocation

5. **Error state handling**
   - Transient errors
   - Persistent errors
   - Recovery strategies
   - Error reporting and logging

## Basic Client Implementation

Here's a basic MCP client implementation in Python:

```python
# Basic MCP client in Python
from mcp.client import MCPClient
import asyncio
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class SimpleMCPClient:
    def __init__(self, server_url: str, auth_token: str = None):
        self.server_url = server_url
        self.auth_token = auth_token
        self.client = MCPClient()
        self.capabilities = {}
        self.connected = False
    
    async def connect(self):
        """Connect to the MCP server"""
        try:
            logger.info(f"Connecting to MCP server at {self.server_url}")
            
            # Set up authentication if token is provided
            if self.auth_token:
                self.client.set_auth_token(self.auth_token)
            
            # Connect to server
            await self.client.connect(self.server_url)
            self.connected = True
            logger.info("Connected to MCP server successfully")
            
            # Discover capabilities
            await self.discover_capabilities()
            
            return True
        except Exception as e:
            logger.error(f"Failed to connect to MCP server: {str(e)}")
            self.connected = False
            return False
    
    async def disconnect(self):
        """Disconnect from the MCP server"""
        if self.connected:
            try:
                await self.client.disconnect()
                logger.info("Disconnected from MCP server")
            except Exception as e:
                logger.error(f"Error disconnecting from MCP server: {str(e)}")
            finally:
                self.connected = False
    
    async def discover_capabilities(self):
        """Discover server capabilities"""
        if not self.connected:
            logger.error("Cannot discover capabilities: Not connected to server")
            return {}
        
        try:
            # Get all capabilities
            capabilities = await self.client.get_capabilities()
            
            # Organize by type
            self.capabilities = {
                "tools": [],
                "resources": [],
                "prompts": []
            }
            
            for cap in capabilities:
                if cap.type == "tool":
                    self.capabilities["tools"].append(cap)
                elif cap.type == "resource":
                    self.capabilities["resources"].append(cap)
                elif cap.type == "prompt":
                    self.capabilities["prompts"].append(cap)
            
            logger.info(f"Discovered {len(capabilities)} capabilities: "
                        f"{len(self.capabilities['tools'])} tools, "
                        f"{len(self.capabilities['resources'])} resources, "
                        f"{len(self.capabilities['prompts'])} prompts")
            
            return self.capabilities
        except Exception as e:
            logger.error(f"Error discovering capabilities: {str(e)}")
            return {}
    
    async def invoke_tool(self, tool_id: str, parameters: dict):
        """Invoke a tool on the server"""
        if not self.connected:
            logger.error("Cannot invoke tool: Not connected to server")
            return None
        
        try:
            logger.info(f"Invoking tool {tool_id} with parameters: {parameters}")
            result = await self.client.invoke_tool(tool_id, parameters)
            logger.info(f"Tool {tool_id} invocation successful")
            return result
        except Exception as e:
            logger.error(f"Error invoking tool {tool_id}: {str(e)}")
            return None
    
    async def get_resource(self, resource_id: str, parameters: dict):
        """Get a resource from the server"""
        if not self.connected:
            logger.error("Cannot get resource: Not connected to server")
            return None
        
        try:
            logger.info(f"Getting resource {resource_id} with parameters: {parameters}")
            result = await self.client.get_resource(resource_id, parameters)
            logger.info(f"Resource {resource_id} retrieval successful")
            return result
        except Exception as e:
            logger.error(f"Error getting resource {resource_id}: {str(e)}")
            return None

# Example usage
async def main():
    # Create client
    client = SimpleMCPClient("http://localhost:8000", "my_auth_token")
    
    # Connect to server
    connected = await client.connect()
    if not connected:
        logger.error("Failed to connect. Exiting.")
        return
    
    try:
        # Invoke a tool
        result = await client.invoke_tool("calculator", {
            "operation": "add",
            "a": 5,
            "b": 3
        })
        
        if result:
            logger.info(f"Calculator result: {result}")
        
        # Get a resource
        resource = await client.get_resource("config", {
            "section": "app"
        })
        
        if resource:
            logger.info(f"Config resource: {resource}")
    
    finally:
        # Disconnect
        await client.disconnect()

# Run the example
if __name__ == "__main__":
    asyncio.run(main())
```

And a similar implementation in TypeScript:

```typescript
// Basic MCP client in TypeScript
import { MCPClient, Capability, ToolResult, ResourceResult } from 'mcp-client';

// Configure logging
const logger = {
  info: (message: string) => console.log(`[INFO] ${message}`),
  error: (message: string) => console.error(`[ERROR] ${message}`)
};

class SimpleMCPClient {
  private serverUrl: string;
  private authToken?: string;
  private client: MCPClient;
  private capabilities: {
    tools: Capability[];
    resources: Capability[];
    prompts: Capability[];
  };
  private connected: boolean;
  
  constructor(serverUrl: string, authToken?: string) {
    this.serverUrl = serverUrl;
    this.authToken = authToken;
    this.client = new MCPClient();
    this.capabilities = {
      tools: [],
      resources: [],
      prompts: []
    };
    this.connected = false;
  }
  
  async connect(): Promise<boolean> {
    try {
      logger.info(`Connecting to MCP server at ${this.serverUrl}`);
      
      // Set up authentication if token is provided
      if (this.authToken) {
        this.client.setAuthToken(this.authToken);
      }
      
      // Connect to server
      await this.client.connect(this.serverUrl);
      this.connected = true;
      logger.info("Connected to MCP server successfully");
      
      // Discover capabilities
      await this.discoverCapabilities();
      
      return true;
    } catch (error) {
      logger.error(`Failed to connect to MCP server: ${error.message}`);
      this.connected = false;
      return false;
    }
  }
  
  async disconnect(): Promise<void> {
    if (this.connected) {
      try {
        await this.client.disconnect();
        logger.info("Disconnected from MCP server");
      } catch (error) {
        logger.error(`Error disconnecting from MCP server: ${error.message}`);
      } finally {
        this.connected = false;
      }
    }
  }
  
  async discoverCapabilities(): Promise<any> {
    if (!this.connected) {
      logger.error("Cannot discover capabilities: Not connected to server");
      return {};
    }
    
    try {
      // Get all capabilities
      const capabilities = await this.client.getCapabilities();
      
      // Reset capabilities
      this.capabilities = {
        tools: [],
        resources: [],
        prompts: []
      };
      
      // Organize by type
      for (const cap of capabilities) {
        if (cap.type === "tool") {
          this.capabilities.tools.push(cap);
        } else if (cap.type === "resource") {
          this.capabilities.resources.push(cap);
        } else if (cap.type === "prompt") {
          this.capabilities.prompts.push(cap);
        }
      }
      
      logger.info(`Discovered ${capabilities.length} capabilities: ` +
                 `${this.capabilities.tools.length} tools, ` +
                 `${this.capabilities.resources.length} resources, ` +
                 `${this.capabilities.prompts.length} prompts`);
      
      return this.capabilities;
    } catch (error) {
      logger.error(`Error discovering capabilities: ${error.message}`);
      return {};
    }
  }
  
  async invokeTool(toolId: string, parameters: any): Promise<ToolResult | null> {
    if (!this.connected) {
      logger.error("Cannot invoke tool: Not connected to server");
      return null;
    }
    
    try {
      logger.info(`Invoking tool ${toolId} with parameters: ${JSON.stringify(parameters)}`);
      const result = await this.client.invokeTool(toolId, parameters);
      logger.info(`Tool ${toolId} invocation successful`);
      return result;
    } catch (error) {
      logger.error(`Error invoking tool ${toolId}: ${error.message}`);
      return null;
    }
  }
  
  async getResource(resourceId: string, parameters: any): Promise<ResourceResult | null> {
    if (!this.connected) {
      logger.error("Cannot get resource: Not connected to server");
      return null;
    }
    
    try {
      logger.info(`Getting resource ${resourceId} with parameters: ${JSON.stringify(parameters)}`);
      const result = await this.client.getResource(resourceId, parameters);
      logger.info(`Resource ${resourceId} retrieval successful`);
      return result;
    } catch (error) {
      logger.error(`Error getting resource ${resourceId}: ${error.message}`);
      return null;
    }
  }
}

// Example usage
async function main() {
  // Create client
  const client = new SimpleMCPClient("http://localhost:8000", "my_auth_token");
  
  // Connect to server
  const connected = await client.connect();
  if (!connected) {
    logger.error("Failed to connect. Exiting.");
    return;
  }
  
  try {
    // Invoke a tool
    const result = await client.invokeTool("calculator", {
      operation: "add",
      a: 5,
      b: 3
    });
    
    if (result) {
      logger.info(`Calculator result: ${JSON.stringify(result)}`);
    }
    
    // Get a resource
    const resource = await client.getResource("config", {
      section: "app"
    });
    
    if (resource) {
      logger.info(`Config resource: ${JSON.stringify(resource)}`);
    }
  } finally {
    // Disconnect
    await client.disconnect();
  }
}

// Run the example
main().catch(error => {
  logger.error(`Unhandled error: ${error.message}`);
});
```

## Error Handling Strategies

Proper error handling is essential for MCP clients:

1. **Connection Errors**
   - Server unavailable
   - Network issues
   - Timeout errors
   - Authentication failures

2. **Protocol Errors**
   - Invalid message format
   - Unsupported protocol version
   - Invalid request
   - Server internal errors

3. **Business Logic Errors**
   - Invalid parameters
   - Resource not found
   - Permission denied
   - Business rule violations

4. **Recovery Strategies**
   - Automatic reconnection
   - Request retry with exponential backoff
   - Fallback to alternative servers
   - Graceful degradation

## Summary

Setting up an MCP client involves choosing an SDK, configuring transport and authentication, and implementing proper connection and state management. The connection process includes transport establishment, version negotiation, authentication, and capability discovery. Proper state management and error handling are essential for robust client implementations.

In the next lesson, we'll explore how to integrate MCP clients with AI models to enable intelligent tool usage.
