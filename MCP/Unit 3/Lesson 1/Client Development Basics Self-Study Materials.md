
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 3/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Practical Exercise 3.1

**Objective**: Create a basic MCP client

### Instructions

1. Choose a programming language and install the appropriate MCP SDK:
   - Python: `pip install mcp-client`
   - Node.js: `npm install @modelcontextprotocol/client`
   - Java: Add the Maven dependency to your `pom.xml`
   - C#: `dotnet add package ModelContextProtocol.Client`

2. Create a simple MCP client that connects to your server from Unit 2:
   - Implement connection establishment
   - Add authentication if your server requires it
   - Handle connection errors and logging

3. Implement the discovery process to retrieve capabilities:
   - Get and display available tools, resources, and prompts
   - Organize capabilities by type
   - Display parameter information for each capability

4. Handle connection errors and reconnection:
   - Implement error detection
   - Add reconnection logic with backoff
   - Log connection events

### Expected Outcome

A functional MCP client that:
- Successfully connects to an MCP server
- Retrieves and displays server capabilities
- Handles connection errors gracefully
- Provides useful logging information

## Code Example (Python)

Here's a starter implementation for a basic MCP client with reconnection logic:

```python
# Basic MCP client with reconnection logic
import asyncio
import logging
import random
from mcp.client import MCPClient

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class MCP_Client:
    def __init__(self, server_url, auth_token=None, max_retries=5):
        self.server_url = server_url
        self.auth_token = auth_token
        self.max_retries = max_retries
        self.client = MCPClient()
        self.capabilities = {}
        self.connected = False
        self.retry_count = 0
    
    async def connect_with_retry(self):
        """Connect to the server with retry logic"""
        while self.retry_count < self.max_retries:
            try:
                if self.retry_count > 0:
                    # Calculate backoff time (exponential with jitter)
                    backoff = min(30, (2 ** self.retry_count) + random.uniform(0, 1))
                    logger.info(f"Retrying connection in {backoff:.2f} seconds (attempt {self.retry_count + 1}/{self.max_retries})")
                    await asyncio.sleep(backoff)
                
                await self.connect()
                
                if self.connected:
                    self.retry_count = 0  # Reset retry count on successful connection
                    return True
                
            except Exception as e:
                logger.error(f"Connection attempt failed: {str(e)}")
            
            self.retry_count += 1
        
        logger.error(f"Failed to connect after {self.max_retries} attempts")
        return False
    
    async def connect(self):
        """Connect to the MCP server"""
        try:
            logger.info(f"Connecting to MCP server at {self.server_url}")
            
            # Set up authentication if provided
            if self.auth_token:
                self.client.set_auth_token(self.auth_token)
            
            # Connect to server with timeout
            await asyncio.wait_for(
                self.client.connect(self.server_url),
                timeout=10
            )
            
            self.connected = True
            logger.info("Connected to MCP server successfully")
            
            # Discover capabilities
            await self.discover_capabilities()
            
            return True
            
        except asyncio.TimeoutError:
            logger.error("Connection timeout")
            self.connected = False
            return False
            
        except Exception as e:
            logger.error(f"Failed to connect: {str(e)}")
            self.connected = False
            return False
    
    async def discover_capabilities(self):
        """Discover and organize server capabilities"""
        if not self.connected:
            logger.warning("Cannot discover capabilities: Not connected")
            return {}
        
        try:
            # Get all capabilities
            all_capabilities = await self.client.get_capabilities()
            
            # Organize capabilities by type
            self.capabilities = {
                "tools": [],
                "resources": [],
                "prompts": []
            }
            
            for capability in all_capabilities:
                if capability.type == "tool":
                    self.capabilities["tools"].append(capability)
                elif capability.type == "resource":
                    self.capabilities["resources"].append(capability)
                elif capability.type == "prompt":
                    self.capabilities["prompts"].append(capability)
            
            # Log discovery results
            logger.info(f"Discovered {len(all_capabilities)} capabilities")
            logger.info(f"  - Tools: {len(self.capabilities['tools'])}")
            logger.info(f"  - Resources: {len(self.capabilities['resources'])}")
            logger.info(f"  - Prompts: {len(self.capabilities['prompts'])}")
            
            return self.capabilities
            
        except Exception as e:
            logger.error(f"Error discovering capabilities: {str(e)}")
            return {}
    
    def display_capabilities(self):
        """Display discovered capabilities in a readable format"""
        if not self.capabilities:
            logger.warning("No capabilities to display")
            return
        
        print("\n=== SERVER CAPABILITIES ===\n")
        
        # Display tools
        if self.capabilities["tools"]:
            print("\n== TOOLS ==\n")
            for tool in self.capabilities["tools"]:
                print(f"Tool: {tool.name} ({tool.id})")
                print(f"  Description: {tool.description}")
                print("  Parameters:")
                for param in tool.parameters:
                    required = "[Required]" if param.required else "[Optional]"
                    default = f", Default: {param.default}" if hasattr(param, 'default') and param.default is not None else ""
                    print(f"    - {param.name} ({param.type}) {required}{default}")
                    print(f"      {param.description}")
                print()
        
        # Display resources
        if self.capabilities["resources"]:
            print("\n== RESOURCES ==\n")
            for resource in self.capabilities["resources"]:
                print(f"Resource: {resource.name} ({resource.id})")
                print(f"  Description: {resource.description}")
                print("  Parameters:")
                for param in resource.parameters:
                    required = "[Required]" if param.required else "[Optional]"
                    default = f", Default: {param.default}" if hasattr(param, 'default') and param.default is not None else ""
                    print(f"    - {param.name} ({param.type}) {required}{default}")
                    print(f"      {param.description}")
                print()
        
        # Display prompts
        if self.capabilities["prompts"]:
            print("\n== PROMPTS ==\n")
            for prompt in self.capabilities["prompts"]:
                print(f"Prompt: {prompt.name} ({prompt.id})")
                print(f"  Description: {prompt.description}")
                print("  Parameters:")
                for param in prompt.parameters:
                    required = "[Required]" if param.required else "[Optional]"
                    default = f", Default: {param.default}" if hasattr(param, 'default') and param.default is not None else ""
                    print(f"    - {param.name} ({param.type}) {required}{default}")
                    print(f"      {param.description}")
                print()
    
    async def disconnect(self):
        """Disconnect from the server"""
        if self.connected:
            try:
                await self.client.disconnect()
                logger.info("Disconnected from MCP server")
            except Exception as e:
                logger.error(f"Error during disconnect: {str(e)}")
            finally:
                self.connected = False

# Example usage
async def main():
    # Create client
    client = MCP_Client(
        server_url="http://localhost:8000",
        auth_token="your_auth_token",  # Optional
        max_retries=3
    )
    
    # Connect with retry logic
    if await client.connect_with_retry():
        # Display capabilities
        client.display_capabilities()
        
        # Disconnect when done
        await client.disconnect()
    else:
        logger.error("Failed to connect to server")

if __name__ == "__main__":
    asyncio.run(main())
```

## Extension Activities

1. **Capability Testing**:
   - Create a test script that verifies each capability
   - Implement automated testing for tool invocation
   - Test error handling for invalid parameters
   - Generate a capability compatibility report

2. **Connection Monitoring**:
   - Implement connection health monitoring
   - Add metrics collection for connection events
   - Create a simple dashboard for connection status
   - Implement alerting for connection issues

3. **Multi-Server Client**:
   - Extend your client to connect to multiple servers
   - Implement server discovery
   - Create a capability registry that merges results
   - Build intelligent routing between servers

## Recommended Reading

1. [MCP Client Implementation Guide](https://modelcontextprotocol.io/docs/guides/client-implementation/)
   - Best practices for client development
   - Error handling strategies
   - Performance optimization techniques

2. [Connection Management Best Practices](https://modelcontextprotocol.io/docs/guides/connection-management/)
   - Connection lifecycle
   - Reconnection patterns
   - Session management
   - Connection pooling

3. [MCP Authentication](https://modelcontextprotocol.io/docs/guides/authentication/)
   - Authentication mechanisms
   - Token management
   - Security best practices
   - OAuth 2.1 integration

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What are the main components needed to set up an MCP client?
2. How does the handshake process work?
3. What information should be retrieved during capability discovery?
4. How should an MCP client handle connection failures?
5. What state should an MCP client maintain?
6. What are the different transport options available for MCP clients?
7. How does authentication work in MCP clients?
8. What strategies can be used for reconnection after failures?

## Connection Analysis Exercise

Analyze different MCP connection scenarios and explain how to handle them:

1. **Initial Connection**:
   - What steps should the client take when first connecting?
   - What information should be exchanged?
   - What errors might occur?
   - How should the client handle successful connection?

2. **Connection Loss**:
   - How can the client detect a lost connection?
   - What strategies should be used for reconnection?
   - How should state be managed during reconnection?
   - What should happen to pending requests?

3. **Authentication Failure**:
   - What causes authentication failures?
   - How should the client handle authentication errors?
   - What recovery strategies are available?
   - How can the client prevent authentication issues?

4. **Server Overload**:
   - How might server overload manifest to the client?
   - What strategies can mitigate impact on the client?
   - How should the client handle rate limiting?
   - What backoff strategies are most effective?

## Client Logging Implementation

Design and implement a comprehensive logging system for your MCP client:

1. Create log categories for:
   - Connection events
   - Authentication
   - Capability discovery
   - Tool invocation
   - Resource retrieval
   - Errors and exceptions

2. Implement log levels:
   - ERROR: For failures requiring immediate attention
   - WARNING: For concerning events that don't prevent operation
   - INFO: For normal operational events
   - DEBUG: For detailed troubleshooting information

3. Include essential information in each log:
   - Timestamp
   - Event category
   - Component identifier
   - Operation details
   - Result status
   - Error details when applicable

## Additional Resources

### Client Development
- [MCP Client SDKs](https://modelcontextprotocol.io/docs/sdks/)
- [Client Architecture Patterns](https://modelcontextprotocol.io/docs/patterns/clients/)
- [Client Testing Strategies](https://modelcontextprotocol.io/docs/guides/client-testing/)

### Connection Management
- [Reconnection Strategies](https://modelcontextprotocol.io/docs/guides/reconnection/)
- [Connection Pooling](https://modelcontextprotocol.io/docs/guides/connection-pooling/)
- [Transport Selection](https://modelcontextprotocol.io/docs/guides/transport-selection/)

### Authentication
- [OAuth 2.1 with MCP](https://modelcontextprotocol.io/docs/guides/oauth/)
- [Token Management](https://modelcontextprotocol.io/docs/guides/token-management/)
- [Client Credentials Security](https://modelcontextprotocol.io/docs/guides/credential-security/)
