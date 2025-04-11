
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 5/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Practical Exercise 5.1

**Objective**: Implement a multi-server MCP client

### Instructions

1. Create a client that connects to at least three different MCP servers:
   - Implement connection management for multiple servers
   - Add server discovery and registration
   - Handle authentication for each server
   - Implement connection health monitoring

2. Implement intelligent routing based on capabilities:
   - Create a routing strategy interface
   - Implement at least two different routing strategies (e.g., round-robin, latency-based)
   - Add metrics collection for routing decisions
   - Create a mechanism to switch between strategies

3. Handle conflicts between similar capabilities:
   - Implement conflict detection for capabilities
   - Create resolution strategies (e.g., version-based, server priority)
   - Add user preference handling
   - Document how conflicts are resolved

4. Create a composite tool that aggregates results from multiple servers:
   - Implement parallel invocation of tools across servers
   - Create result aggregation logic
   - Handle partial failures
   - Present unified results to the user

### Expected Outcome

A multi-server MCP client that:
- Successfully connects to multiple servers
- Routes requests intelligently based on capabilities
- Handles capability conflicts effectively
- Aggregates results from multiple servers

## Code Example (Multi-Server Client)

Here's a simplified starter implementation for a multi-server client:

```python
# Simple multi-server MCP client
import asyncio
import logging
from typing import Dict, List, Any, Optional

from mcp.client import MCPClient
from mcp.schemas import MCPCapability

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class SimpleMultiServerClient:
    """A simple client for managing multiple MCP servers"""
    
    def __init__(self):
        # Store clients by ID
        self.clients = {}
        
        # Store server information
        self.servers = {}
        
        # Store capabilities
        self.capabilities = {}
        
        # Connected servers
        self.connected = []
    
    async def add_server(self, server_id: str, url: str, auth_token: Optional[str] = None) -> bool:
        """Add a new server to the client"""
        # Create new client
        client = MCPClient()
        
        # Set authentication if provided
        if auth_token:
            client.set_auth_token(auth_token)
        
        # Store client and server info
        self.clients[server_id] = client
        self.servers[server_id] = {
            "id": server_id,
            "url": url,
            "status": "disconnected"
        }
        
        # Try to connect
        return await self.connect(server_id)
    
    async def connect(self, server_id: str) -> bool:
        """Connect to a server"""
        if server_id not in self.clients:
            logger.error(f"Unknown server: {server_id}")
            return False
        
        if server_id in self.connected:
            logger.info(f"Server {server_id} already connected")
            return True
        
        client = self.clients[server_id]
        url = self.servers[server_id]["url"]
        
        try:
            # Connect to server
            await client.connect(url)
            
            # Update status
            self.servers[server_id]["status"] = "connected"
            self.connected.append(server_id)
            
            # Discover capabilities
            capabilities = await client.get_capabilities()
            
            # Store capabilities
            for capability in capabilities:
                cap_id = capability.id
                
                if cap_id not in self.capabilities:
                    self.capabilities[cap_id] = []
                
                self.capabilities[cap_id].append((server_id, capability))
            
            logger.info(f"Connected to {server_id} with {len(capabilities)} capabilities")
            return True
        
        except Exception as e:
            logger.error(f"Failed to connect to {server_id}: {str(e)}")
            self.servers[server_id]["status"] = "error"
            self.servers[server_id]["error"] = str(e)
            return False
    
    async def disconnect(self, server_id: str) -> bool:
        """Disconnect from a server"""
        if server_id not in self.clients:
            logger.error(f"Unknown server: {server_id}")
            return False
        
        if server_id not in self.connected:
            logger.info(f"Server {server_id} already disconnected")
            return True
        
        client = self.clients[server_id]
        
        try:
            # Disconnect from server
            await client.disconnect()
            
            # Update status
            self.servers[server_id]["status"] = "disconnected"
            self.connected.remove(server_id)
            
            # Remove capabilities
            for cap_id in list(self.capabilities.keys()):
                self.capabilities[cap_id] = [
                    (s_id, cap) for s_id, cap in self.capabilities[cap_id]
                    if s_id != server_id
                ]
                
                # Remove capability if no servers provide it
                if not self.capabilities[cap_id]:
                    del self.capabilities[cap_id]
            
            logger.info(f"Disconnected from {server_id}")
            return True
        
        except Exception as e:
            logger.error(f"Failed to disconnect from {server_id}: {str(e)}")
            return False
    
    def get_servers_for_capability(self, capability_id: str) -> List[str]:
        """Get servers that provide a capability"""
        if capability_id not in self.capabilities:
            return []
        
        return [server_id for server_id, _ in self.capabilities[capability_id]]
    
    def get_best_server_for_capability(self, capability_id: str) -> Optional[str]:
        """Get the best server for a capability (simple implementation)"""
        servers = self.get_servers_for_capability(capability_id)
        
        if not servers:
            return None
        
        # Simple strategy: return first server
        return servers[0]
    
    async def invoke_tool(self, tool_id: str, parameters: dict) -> Any:
        """Invoke a tool on the best server"""
        server_id = self.get_best_server_for_capability(tool_id)
        
        if not server_id:
            raise Exception(f"No server provides tool: {tool_id}")
        
        client = self.clients[server_id]
        
        try:
            logger.info(f"Invoking {tool_id} on {server_id}")
            return await client.invoke_tool(tool_id, parameters)
        except Exception as e:
            logger.error(f"Failed to invoke {tool_id} on {server_id}: {str(e)}")
            raise
    
    async def get_resource(self, resource_id: str, parameters: dict) -> Any:
        """Get a resource from the best server"""
        server_id = self.get_best_server_for_capability(resource_id)
        
        if not server_id:
            raise Exception(f"No server provides resource: {resource_id}")
        
        client = self.clients[server_id]
        
        try:
            logger.info(f"Getting {resource_id} from {server_id}")
            return await client.get_resource(resource_id, parameters)
        except Exception as e:
            logger.error(f"Failed to get {resource_id} from {server_id}: {str(e)}")
            raise

# Example usage
async def main():
    client = SimpleMultiServerClient()
    
    # Add servers
    await client.add_server("weather", "http://localhost:8001")
    await client.add_server("search", "http://localhost:8002")
    await client.add_server("calendar", "http://localhost:8003")
    
    # Invoke a tool
    try:
        weather = await client.invoke_tool("weather.forecast", {
            "location": "New York",
            "days": 3
        })
        print(f"Weather forecast: {weather}")
    except Exception as e:
        print(f"Error getting weather: {e}")
    
    # Disconnect from all servers
    for server_id in list(client.connected):
        await client.disconnect(server_id)

if __name__ == "__main__":
    asyncio.run(main())
```

## Extension Activities

1. **Advanced Routing Strategy**:
   - Implement a machine learning-based routing strategy
   - Train the strategy using historical performance data
   - Add real-time adaptation based on server performance
   - Create a visualization of routing decisions

2. **Distributed Capability Registry**:
   - Implement a central registry for capabilities
   - Add dynamic server discovery
   - Create health monitoring for servers
   - Implement automated failover

3. **Multi-tier MCP Architecture**:
   - Design a hierarchical MCP server architecture
   - Implement proxy servers that aggregate capabilities
   - Create specialized servers for different domains
   - Implement cross-tier orchestration

## Recommended Reading

1. [MCP Multi-Server Management](https://modelcontextprotocol.io/docs/guides/multi-server/)
   - Best practices for multi-server architecture
   - Connection management strategies
   - Server discovery patterns
   - Authentication coordination

2. [Capability Routing Strategies](https://modelcontextprotocol.io/docs/guides/capability-routing/)
   - Routing algorithm design
   - Load balancing techniques
   - Latency optimization
   - Geographic routing considerations

3. [Distributed Systems Patterns](https://modelcontextprotocol.io/docs/patterns/distributed-systems/)
   - Service discovery
   - Circuit breaker pattern
   - Bulkhead pattern
   - Sidecar pattern

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What challenges arise when managing multiple MCP servers?
2. How should requests be routed across servers?
3. What strategies can resolve capability conflicts?
4. How can results from multiple servers be effectively aggregated?
5. What fallback mechanisms should be implemented in multi-server environments?
6. What design patterns are useful for multi-server MCP architectures?
7. How does authentication work across multiple servers?
8. What monitoring considerations are important for multi-server deployments?

## Routing Strategy Design

Design a sophisticated routing strategy for a multi-server environment:

1. **Strategy Interface**:
   - Define a clear interface for routing strategies
   - Determine what metrics are needed for decision-making
   - Create a mechanism to update metrics
   - Design a feedback mechanism for routing decisions

2. **Algorithm Selection**:
   - Choose appropriate algorithms for different scenarios (e.g., response time, load, geographic)
   - Implement weighted decision-making
   - Consider machine learning approaches
   - Design for adaptation and learning

3. **Implementation Plan**:
   - Outline how the strategy would be implemented
   - Determine how metrics would be collected
   - Design monitoring and visualization
   - Create a testing approach

Document your design with:
- Interface definitions
- Algorithm descriptions
- Decision flow diagrams
- Implementation considerations

## Architecture Diagram

Create an architecture diagram for a multi-server MCP environment:

1. Include components for:
   - MCP Clients
   - Load Balancer/Router
   - Multiple MCP Servers
   - Authentication Service
   - Capability Registry
   - Monitoring System

2. Show the flow of:
   - Connection Establishment
   - Capability Discovery
   - Request Routing
   - Result Aggregation
   - Error Handling

3. Annotate the diagram with:
   - Key design decisions
   - Security considerations
   - Scaling aspects
   - Failure modes

## Additional Resources

### Design Patterns
- [Service Registry Pattern](https://modelcontextprotocol.io/docs/patterns/service-registry/)
- [API Gateway Pattern](https://modelcontextprotocol.io/docs/patterns/api-gateway/)
- [Circuit Breaker Pattern](https://modelcontextprotocol.io/docs/patterns/circuit-breaker/)
- [Bulkhead Pattern](https://modelcontextprotocol.io/docs/patterns/bulkhead/)

### Implementation Techniques
- [Connection Pooling](https://modelcontextprotocol.io/docs/techniques/connection-pooling/)
- [Load Balancing Algorithms](https://modelcontextprotocol.io/docs/techniques/load-balancing/)
- [Distributed Authentication](https://modelcontextprotocol.io/docs/techniques/distributed-auth/)
- [Result Aggregation Methods](https://modelcontextprotocol.io/docs/techniques/result-aggregation/)

### Case Studies
- [Multi-Server MCP in Enterprise](https://modelcontextprotocol.io/case-studies/enterprise-mcp/)
- [Global MCP Deployment](https://modelcontextprotocol.io/case-studies/global-mcp/)
- [High-Availability MCP Architecture](https://modelcontextprotocol.io/case-studies/ha-mcp/)
