
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 5/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Overview

This lesson explores managing connections to multiple MCP servers and orchestrating capabilities across them.

## Managing Connections to Multiple MCP Servers

Multi-server environments require:

1. **Connection management**
   - Establishing multiple connections
   - Managing connection lifecycles
   - Monitoring connection health
   - Resource allocation

2. **Server discovery**
   - Manual configuration
   - Automatic discovery
   - Registry-based lookup
   - Dynamic server addition

3. **Authentication coordination**
   - Shared credentials
   - Per-server authentication
   - Token management
   - Single sign-on integration

4. **Health monitoring**
   - Connection status
   - Response times
   - Error rates
   - Load balancing metrics

5. **Failover strategies**
   - Connection retry
   - Server switching
   - Graceful degradation
   - High availability patterns

Effective management of multiple MCP server connections enables more robust and flexible AI applications.

## Multi-Server Client Implementation

Here's an example of a client that manages connections to multiple MCP servers:

```python
# Multi-server MCP client
import asyncio
import logging
import random
import time
from typing import Dict, List, Any, Optional, Tuple

from mcp.client import MCPClient
from mcp.schemas import MCPCapability

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class MultiServerMCPClient:
    """Client for managing multiple MCP server connections"""
    
    def __init__(self):
        # Store clients by server ID
        self.clients: Dict[str, MCPClient] = {}
        
        # Store server information
        self.servers: Dict[str, dict] = {}
        
        # Cache capabilities by ID and server
        self.capabilities: Dict[str, List[Tuple[str, MCPCapability]]] = {}
        
        # Track connection status
        self.connected_servers: List[str] = []
    
    async def add_server(self, server_id: str, server_url: str, auth_token: Optional[str] = None) -> bool:
        """Add a new server to the client"""
        if server_id in self.servers:
            logger.warning(f"Server ID {server_id} already exists")
            return False
        
        # Create new client
        client = MCPClient()
        
        # Set authentication if provided
        if auth_token:
            client.set_auth_token(auth_token)
        
        # Store server information
        self.servers[server_id] = {
            "id": server_id,
            "url": server_url,
            "auth_token": auth_token,
            "status": "disconnected",
            "last_connected": None,
            "added_at": time.time()
        }
        
        # Store client
        self.clients[server_id] = client
        
        # Attempt to connect
        return await self.connect_server(server_id)
    
    async def connect_server(self, server_id: str) -> bool:
        """Connect to a specific server"""
        if server_id not in self.servers:
            logger.error(f"Server ID {server_id} not found")
            return False
        
        if server_id in self.connected_servers:
            logger.info(f"Server {server_id} is already connected")
            return True
        
        client = self.clients[server_id]
        server_url = self.servers[server_id]["url"]
        
        try:
            logger.info(f"Connecting to server {server_id} at {server_url}")
            
            # Connect to server
            await client.connect(server_url)
            
            # Update server status
            self.servers[server_id]["status"] = "connected"
            self.servers[server_id]["last_connected"] = time.time()
            self.connected_servers.append(server_id)
            
            # Discover capabilities
            await self.discover_capabilities(server_id)
            
            logger.info(f"Successfully connected to server {server_id}")
            return True
            
        except Exception as e:
            logger.error(f"Failed to connect to server {server_id}: {str(e)}")
            self.servers[server_id]["status"] = "error"
            self.servers[server_id]["error"] = str(e)
            return False
    
    async def disconnect_server(self, server_id: str) -> bool:
        """Disconnect from a specific server"""
        if server_id not in self.servers:
            logger.error(f"Server ID {server_id} not found")
            return False
        
        if server_id not in self.connected_servers:
            logger.info(f"Server {server_id} is already disconnected")
            return True
        
        client = self.clients[server_id]
        
        try:
            logger.info(f"Disconnecting from server {server_id}")
            
            # Disconnect from server
            await client.disconnect()
            
            # Update server status
            self.servers[server_id]["status"] = "disconnected"
            self.connected_servers.remove(server_id)
            
            # Clear capabilities for this server
            for cap_id, servers in self.capabilities.items():
                self.capabilities[cap_id] = [
                    (s_id, cap) for s_id, cap in servers if s_id != server_id
                ]
            
            logger.info(f"Successfully disconnected from server {server_id}")
            return True
            
        except Exception as e:
            logger.error(f"Failed to disconnect from server {server_id}: {str(e)}")
            return False
    
    async def connect_all_servers(self) -> Dict[str, bool]:
        """Connect to all servers"""
        results = {}
        
        for server_id in self.servers:
            results[server_id] = await self.connect_server(server_id)
        
        return results
    
    async def disconnect_all_servers(self) -> Dict[str, bool]:
        """Disconnect from all servers"""
        results = {}
        
        for server_id in list(self.connected_servers):
            results[server_id] = await self.disconnect_server(server_id)
        
        return results
    
    async def discover_capabilities(self, server_id: str) -> List[MCPCapability]:
        """Discover capabilities for a specific server"""
        if server_id not in self.connected_servers:
            logger.error(f"Server {server_id} is not connected")
            return []
        
        client = self.clients[server_id]
        
        try:
            logger.info(f"Discovering capabilities for server {server_id}")
            
            # Get capabilities
            capabilities = await client.get_capabilities()
            
            # Store capabilities
            for capability in capabilities:
                cap_id = capability.id
                
                if cap_id not in self.capabilities:
                    self.capabilities[cap_id] = []
                
                # Check if capability already exists for this server
                exists = any(s_id == server_id for s_id, _ in self.capabilities[cap_id])
                
                if not exists:
                    self.capabilities[cap_id].append((server_id, capability))
            
            logger.info(f"Discovered {len(capabilities)} capabilities for server {server_id}")
            return capabilities
            
        except Exception as e:
            logger.error(f"Failed to discover capabilities for server {server_id}: {str(e)}")
            return []
    
    def get_all_capabilities(self) -> List[Tuple[str, MCPCapability]]:
        """Get all capabilities from all servers"""
        all_capabilities = []
        
        for cap_list in self.capabilities.values():
            all_capabilities.extend(cap_list)
        
        return all_capabilities
    
    def get_capability_servers(self, capability_id: str) -> List[Tuple[str, MCPCapability]]:
        """Get all servers that provide a specific capability"""
        return self.capabilities.get(capability_id, [])
    
    async def invoke_tool(self, tool_id: str, parameters: dict, server_id: Optional[str] = None) -> Any:
        """Invoke a tool on a specific server or automatically select server"""
        # Find servers with this tool
        servers_with_tool = self.get_capability_servers(tool_id)
        
        if not servers_with_tool:
            raise Exception(f"Tool {tool_id} not found on any connected server")
        
        # If server ID provided, use that server
        if server_id:
            # Check if server has the tool
            matching_servers = [(s_id, cap) for s_id, cap in servers_with_tool if s_id == server_id]
            
            if not matching_servers:
                raise Exception(f"Tool {tool_id} not found on server {server_id}")
            
            # Get client for server
            client = self.clients[server_id]
            
            # Invoke tool
            logger.info(f"Invoking tool {tool_id} on specified server {server_id}")
            return await client.invoke_tool(tool_id, parameters)
        
        # Otherwise, select a server
        # For this example, we'll just use the first available server
        # In a real implementation, this could use load balancing, etc.
        selected_server_id, _ = servers_with_tool[0]
        client = self.clients[selected_server_id]
        
        # Invoke tool
        logger.info(f"Invoking tool {tool_id} on automatically selected server {selected_server_id}")
        return await client.invoke_tool(tool_id, parameters)
    
    async def get_resource(self, resource_id: str, parameters: dict, server_id: Optional[str] = None) -> Any:
        """Get a resource from a specific server or automatically select server"""
        # Find servers with this resource
        servers_with_resource = self.get_capability_servers(resource_id)
        
        if not servers_with_resource:
            raise Exception(f"Resource {resource_id} not found on any connected server")
        
        # If server ID provided, use that server
        if server_id:
            # Check if server has the resource
            matching_servers = [(s_id, cap) for s_id, cap in servers_with_resource if s_id == server_id]
            
            if not matching_servers:
                raise Exception(f"Resource {resource_id} not found on server {server_id}")
            
            # Get client for server
            client = self.clients[server_id]
            
            # Get resource
            logger.info(f"Getting resource {resource_id} from specified server {server_id}")
            return await client.get_resource(resource_id, parameters)
        
        # Otherwise, select a server
        selected_server_id, _ = servers_with_resource[0]
        client = self.clients[selected_server_id]
        
        # Get resource
        logger.info(f"Getting resource {resource_id} from automatically selected server {selected_server_id}")
        return await client.get_resource(resource_id, parameters)
    
    def get_server_status(self, server_id: str) -> dict:
        """Get status information for a specific server"""
        if server_id not in self.servers:
            raise Exception(f"Server {server_id} not found")
        
        server_info = self.servers[server_id].copy()
        
        # Add capability counts
        tool_count = 0
        resource_count = 0
        prompt_count = 0
        
        for cap_list in self.capabilities.values():
            for s_id, cap in cap_list:
                if s_id == server_id:
                    if cap.type == "tool":
                        tool_count += 1
                    elif cap.type == "resource":
                        resource_count += 1
                    elif cap.type == "prompt":
                        prompt_count += 1
        
        server_info["capabilities"] = {
            "tools": tool_count,
            "resources": resource_count,
            "prompts": prompt_count,
            "total": tool_count + resource_count + prompt_count
        }
        
        return server_info
    
    def get_all_server_status(self) -> Dict[str, dict]:
        """Get status information for all servers"""
        return {server_id: self.get_server_status(server_id) for server_id in self.servers}
```

## Routing Requests Across Servers

Effective routing strategies:

1. **Capability-based routing**
   - Route based on capability availability
   - Multi-capability coordination
   - Capability version matching
   - Capability preferences

2. **Load balancing**
   - Round-robin distribution
   - Least-connection routing
   - Response time optimization
   - Server load consideration

3. **Latency optimization**
   - Geographic proximity
   - Network performance
   - Server response time
   - Caching strategies

4. **Geographic distribution**
   - Data sovereignty compliance
   - Regional server selection
   - Cross-region replication
   - Local data prioritization

5. **Fallback mechanisms**
   - Primary/backup strategies
   - Service degradation handling
   - Request retry policies
   - Alternative capability selection

Here's an example of implementing capability-based routing:

```python
# Capability-based routing extension for multi-server MCP client
import random
from typing import List, Dict, Any, Optional, Tuple

class RoutingStrategy:
    """Base class for routing strategies"""
    
    async def select_server(self, client, capability_id: str, parameters: dict) -> str:
        """Select a server for a capability"""
        raise NotImplementedError
    
    def update_metrics(self, server_id: str, capability_id: str, execution_time: float, success: bool):
        """Update metrics for a server and capability"""
        pass

class RandomRoutingStrategy(RoutingStrategy):
    """Simple random routing strategy"""
    
    async def select_server(self, client, capability_id: str, parameters: dict) -> str:
        """Select a random server for a capability"""
        servers = client.get_capability_servers(capability_id)
        
        if not servers:
            raise Exception(f"Capability {capability_id} not found on any connected server")
        
        # Just pick a random server
        server_id, _ = random.choice(servers)
        return server_id

class RoundRobinRoutingStrategy(RoutingStrategy):
    """Round-robin routing strategy"""
    
    def __init__(self):
        self.last_server_index = {}
    
    async def select_server(self, client, capability_id: str, parameters: dict) -> str:
        """Select the next server in round-robin fashion"""
        servers = client.get_capability_servers(capability_id)
        
        if not servers:
            raise Exception(f"Capability {capability_id} not found on any connected server")
        
        # Get the last index used for this capability
        last_index = self.last_server_index.get(capability_id, -1)
        
        # Get the next index
        next_index = (last_index + 1) % len(servers)
        
        # Update the last index
        self.last_server_index[capability_id] = next_index
        
        # Return the server ID
        server_id, _ = servers[next_index]
        return server_id

class LatencyBasedRoutingStrategy(RoutingStrategy):
    """Latency-based routing strategy"""
    
    def __init__(self):
        self.latency_metrics = {}
        self.request_counts = {}
    
    async def select_server(self, client, capability_id: str, parameters: dict) -> str:
        """Select the server with the lowest latency"""
        servers = client.get_capability_servers(capability_id)
        
        if not servers:
            raise Exception(f"Capability {capability_id} not found on any connected server")
        
        # If we don't have metrics for all servers, use round-robin
        if not all(f"{s_id}:{capability_id}" in self.latency_metrics for s_id, _ in servers):
            # Fall back to round-robin for initial selection
            strategy = RoundRobinRoutingStrategy()
            return await strategy.select_server(client, capability_id, parameters)
        
        # Find the server with the lowest latency
        best_server_id = None
        best_latency = float('inf')
        
        for server_id, _ in servers:
            key = f"{server_id}:{capability_id}"
            latency = self.latency_metrics.get(key, float('inf'))
            
            if latency < best_latency:
                best_latency = latency
                best_server_id = server_id
        
        return best_server_id
    
    def update_metrics(self, server_id: str, capability_id: str, execution_time: float, success: bool):
        """Update latency metrics for a server and capability"""
        key = f"{server_id}:{capability_id}"
        
        # Update request count
        self.request_counts[key] = self.request_counts.get(key, 0) + 1
        
        # Skip failed requests for latency calculation
        if not success:
            return
        
        # Update latency with exponential moving average
        alpha = 0.2  # Weight for new data
        current = self.latency_metrics.get(key, execution_time)
        self.latency_metrics[key] = (alpha * execution_time) + ((1 - alpha) * current)

class LoadBalancedRoutingStrategy(RoutingStrategy):
    """Load-balanced routing strategy"""
    
    def __init__(self):
        self.active_requests = {}
    
    async def select_server(self, client, capability_id: str, parameters: dict) -> str:
        """Select the server with the lowest load"""
        servers = client.get_capability_servers(capability_id)
        
        if not servers:
            raise Exception(f"Capability {capability_id} not found on any connected server")
        
        # Find the server with the lowest number of active requests
        best_server_id = None
        lowest_load = float('inf')
        
        for server_id, _ in servers:
            load = self.active_requests.get(server_id, 0)
            
            if load < lowest_load:
                lowest_load = load
                best_server_id = server_id
        
        # Increment active requests for selected server
        self.active_requests[best_server_id] = self.active_requests.get(best_server_id, 0) + 1
        
        return best_server_id
    
    def update_metrics(self, server_id: str, capability_id: str, execution_time: float, success: bool):
        """Update server load metrics"""
        # Decrement active requests for server
        self.active_requests[server_id] = max(0, self.active_requests.get(server_id, 1) - 1)

# Enhanced multi-server client with routing
class RoutingMultiServerMCPClient(MultiServerMCPClient):
    """Multi-server MCP client with request routing"""
    
    def __init__(self, routing_strategy: Optional[RoutingStrategy] = None):
        super().__init__()
        self.routing_strategy = routing_strategy or RoundRobinRoutingStrategy()
    
    def set_routing_strategy(self, strategy: RoutingStrategy):
        """Set the routing strategy"""
        self.routing_strategy = strategy
    
    async def invoke_tool(self, tool_id: str, parameters: dict, server_id: Optional[str] = None) -> Any:
        """Invoke a tool with routing"""
        # If server ID specified, use parent method
        if server_id:
            return await super().invoke_tool(tool_id, parameters, server_id)
        
        # Use routing strategy to select server
        selected_server_id = await self.routing_strategy.select_server(self, tool_id, parameters)
        
        # Get client for server
        client = self.clients[selected_server_id]
        
        # Track start time
        start_time = time.time()
        success = False
        
        try:
            # Invoke tool
            logger.info(f"Invoking tool {tool_id} on routed server {selected_server_id}")
            result = await client.invoke_tool(tool_id, parameters)
            
            # Track success
            success = True
            
            return result
            
        finally:
            # Calculate execution time
            execution_time = time.time() - start_time
            
            # Update metrics
            self.routing_strategy.update_metrics(selected_server_id, tool_id, execution_time, success)
```

## Conflict Resolution Strategies

Handling capability conflicts:

1. **Naming collision resolution**
   - Namespace prefixing
   - Server-specific prefixes
   - Capability versioning
   - Conflict detection

2. **Version management**
   - Version compatibility checking
   - Version negotiation
   - Feature detection
   - Backward compatibility

3. **Capability prioritization**
   - Server priority ranking
   - Capability quality metrics
   - User preference integration
   - Context-based selection

4. **User preference management**
   - Default server preferences
   - Capability-specific preferences
   - Context-based overrides
   - User control options

5. **Transparent handling**
   - Conflict logging
   - Automatic resolution
   - User notification
   - Selection justification

Here's an example of implementing conflict resolution:

```python
# Conflict resolution for multi-server MCP client
from enum import Enum
from typing import Dict, List, Any, Optional, Tuple

class ConflictResolutionStrategy(Enum):
    """Strategies for resolving capability conflicts"""
    FIRST_SERVER = "first_server"  # Use the first server's capability
    LAST_SERVER = "last_server"  # Use the last server's capability
    NEWEST_VERSION = "newest_version"  # Use the capability with the newest version
    OLDEST_VERSION = "oldest_version"  # Use the capability with the oldest version
    SERVER_PRIORITY = "server_priority"  # Use capability from highest priority server
    USER_PREFERENCE = "user_preference"  # Use user's preferred capability
    MERGE = "merge"  # Attempt to merge capabilities

class CapabilityConflictResolver:
    """Resolver for capability conflicts"""
    
    def __init__(self, default_strategy: ConflictResolutionStrategy = ConflictResolutionStrategy.FIRST_SERVER):
        self.default_strategy = default_strategy
        self.server_priorities = {}
        self.user_preferences = {}
        self.capability_strategies = {}
    
    def set_server_priority(self, server_id: str, priority: int):
        """Set priority for a server (higher number = higher priority)"""
        self.server_priorities[server_id] = priority
    
    def set_user_preference(self, capability_id: str, server_id: str):
        """Set user preference for a capability"""
        self.user_preferences[capability_id] = server_id
    
    def set_capability_strategy(self, capability_id: str, strategy: ConflictResolutionStrategy):
        """Set conflict resolution strategy for a specific capability"""
        self.capability_strategies[capability_id] = strategy
    
    def resolve_conflict(self, capability_id: str, capabilities: List[Tuple[str, MCPCapability]]) -> Tuple[str, MCPCapability]:
        """Resolve conflict for a capability"""
        if len(capabilities) <= 1:
            return capabilities[0]
        
        # Get strategy for capability
        strategy = self.capability_strategies.get(capability_id, self.default_strategy)
        
        if strategy == ConflictResolutionStrategy.FIRST_SERVER:
            return capabilities[0]
        
        elif strategy == ConflictResolutionStrategy.LAST_SERVER:
            return capabilities[-1]
        
        elif strategy == ConflictResolutionStrategy.NEWEST_VERSION:
            # Find capability with newest version
            best_version = "0.0.0"
            best_capability = capabilities[0]
            
            for server_id, capability in capabilities:
                if hasattr(capability, "version") and capability.version:
                    if self._compare_versions(capability.version, best_version) > 0:
                        best_version = capability.version
                        best_capability = (server_id, capability)
            
            return best_capability
        
        elif strategy == ConflictResolutionStrategy.OLDEST_VERSION:
            # Find capability with oldest version
            best_version = "999.999.999"
            best_capability = capabilities[0]
            
            for server_id, capability in capabilities:
                if hasattr(capability, "version") and capability.version:
                    if self._compare_versions(capability.version, best_version) < 0:
                        best_version = capability.version
                        best_capability = (server_id, capability)
            
            return best_capability
        
        elif strategy == ConflictResolutionStrategy.SERVER_PRIORITY:
            # Find capability from highest priority server
            highest_priority = -1
            best_capability = capabilities[0]
            
            for server_id, capability in capabilities:
                priority = self.server_priorities.get(server_id, 0)
                
                if priority > highest_priority:
                    highest_priority = priority
                    best_capability = (server_id, capability)
            
            return best_capability
        
        elif strategy == ConflictResolutionStrategy.USER_PREFERENCE:
            # Check if user has preference for this capability
            preferred_server_id = self.user_preferences.get(capability_id)
            
            if preferred_server_id:
                # Find capability from preferred server
                for server_id, capability in capabilities:
                    if server_id == preferred_server_id:
                        return (server_id, capability)
            
            # Fall back to default strategy
            return self.resolve_conflict(capability_id, capabilities)
        
        elif strategy == ConflictResolutionStrategy.MERGE:
            # Attempt to merge capabilities
            # This is a simplified example - real merging would be more complex
            merged_server_id = "merged"
            merged_capability = capabilities[0][1].copy()
            
            # For this example, we'll just take the union of parameters
            merged_parameters = []
            parameter_names = set()
            
            for server_id, capability in capabilities:
                for param in capability.parameters:
                    if param.name not in parameter_names:
                        merged_parameters.append(param)
                        parameter_names.add(param.name)
            
            merged_capability.parameters = merged_parameters
            
            return (merged_server_id, merged_capability)
        
        # Default to first server
        return capabilities[0]
    
    def _compare_versions(self, version1: str, version2: str) -> int:
        """Compare two version strings"""
        v1_parts = [int(x) for x in version1.split('.')]
        v2_parts = [int(x) for x in version2.split('.')]
        
        # Pad with zeros if necessary
        while len(v1_parts) < 3:
            v1_parts.append(0)
        while len(v2_parts) < 3:
            v2_parts.append(0)
        
        # Compare version components
        for i in range(max(len(v1_parts), len(v2_parts))):
            v1 = v1_parts[i] if i < len(v1_parts) else 0
            v2 = v2_parts[i] if i < len(v2_parts) else 0
            
            if v1 < v2:
                return -1
            elif v1 > v2:
                return 1
        
        return 0

# Enhanced multi-server client with conflict resolution
class ConflictResolvingMultiServerMCPClient(RoutingMultiServerMCPClient):
    """Multi-server MCP client with conflict resolution"""
    
    def __init__(self, routing_strategy: Optional[RoutingStrategy] = None, 
                 conflict_resolver: Optional[CapabilityConflictResolver] = None):
        super().__init__(routing_strategy)
        self.conflict_resolver = conflict_resolver or CapabilityConflictResolver()
    
    async def discover_capabilities(self, server_id: str) -> List[MCPCapability]:
        """Discover capabilities with conflict resolution"""
        capabilities = await super().discover_capabilities(server_id)
        
        # After discovery, resolve any conflicts
        self._resolve_capability_conflicts()
        
        return capabilities
    
    def _resolve_capability_conflicts(self):
        """Resolve conflicts for all capabilities"""
        for capability_id, servers_with_capability in self.capabilities.items():
            if len(servers_with_capability) > 1:
                # We have a conflict
                logger.info(f"Resolving conflict for capability {capability_id} across {len(servers_with_capability)} servers")
                
                # Resolve conflict
                resolved_server_id, resolved_capability = self.conflict_resolver.resolve_conflict(
                    capability_id, servers_with_capability
                )
                
                # If server ID is "merged", this is a special case
                if resolved_server_id == "merged":
                    logger.info(f"Using merged capability for {capability_id}")
                    
                    # We keep all servers but update their capabilities
                    for i in range(len(servers_with_capability)):
                        server_id, _ = servers_with_capability[i]
                        servers_with_capability[i] = (server_id, resolved_capability)
                else:
                    # Replace capability list with just the resolved one
                    self.capabilities[capability_id] = [(resolved_server_id, resolved_capability)]
                    
                    logger.info(f"Resolved conflict for {capability_id} by selecting server {resolved_server_id}")
```

## Composite Results and Aggregation

Combining results from multiple servers:

1. **Data fusion techniques**
   - Result merging
   - Complementary data combination
   - Redundancy elimination
   - Consistency checking

2. **Result normalization**
   - Schema standardization
   - Value normalization
   - Unit conversion
   - Format consistency

3. **Confidence scoring**
   - Result reliability estimation
   - Source quality assessment
   - Freshness evaluation
   - Consistency measurement

4. **Conflict resolution**
   - Value conflict detection
   - Resolution strategies
   - User preference application
   - Majority voting

5. **Presentation strategies**
   - Unified result view
   - Source attribution
   - Confidence indication
   - Alternative presentation

Here's an example of implementing result aggregation:

```python
# Result aggregation for multi-server MCP client
from typing import Dict, List, Any, Optional, Tuple
import asyncio

class ResultAggregator:
    """Aggregator for results from multiple servers"""
    
    async def aggregate_tool_results(self, client, tool_id: str, parameters: dict, strategy: str = "all", timeout: float = 10.0) -> Any:
        """Aggregate results from multiple servers for a tool"""
        servers = client.get_capability_servers(tool_id)
        
        if not servers:
            raise Exception(f"Tool {tool_id} not found on any connected server")
        
        if strategy == "first":
            # Just get result from first available server
            server_id, _ = servers[0]
            return await client.invoke_tool(tool_id, parameters, server_id)
        
        elif strategy == "best":
            # Use the routing strategy to pick the best server
            server_id = await client.routing_strategy.select_server(client, tool_id, parameters)
            return await client.invoke_tool(tool_id, parameters, server_id)
        
        elif strategy == "all":
            # Get results from all servers
            tasks = []
            
            for server_id, _ in servers:
                task = asyncio.create_task(self._invoke_tool_with_timeout(
                    client, tool_id, parameters, server_id, timeout
                ))
                tasks.append((server_id, task))
            
            # Wait for all tasks to complete or timeout
            results = {}
            
            for server_id, task in tasks:
                try:
                    result = await task
                    results[server_id] = {
                        "status": "success",
                        "result": result
                    }
                except asyncio.TimeoutError:
                    results[server_id] = {
                        "status": "timeout",
                        "error": "Request timed out"
                    }
                except Exception as e:
                    results[server_id] = {
                        "status": "error",
                        "error": str(e)
                    }
            
            # Aggregate results based on tool type
            return self._aggregate_results(tool_id, results)
        
        else:
            raise ValueError(f"Unknown aggregation strategy: {strategy}")
    
    async def _invoke_tool_with_timeout(self, client, tool_id: str, parameters: dict, server_id: str, timeout: float) -> Any:
        """Invoke a tool with timeout"""
        return await asyncio.wait_for(
            client.invoke_tool(tool_id, parameters, server_id),
            timeout
        )
    
    def _aggregate_results(self, tool_id: str, results: Dict[str, dict]) -> Any:
        """Aggregate results based on tool type"""
        # Count successful results
        successful_results = {
            server_id: result["result"]
            for server_id, result in results.items()
            if result["status"] == "success"
        }
        
        if not successful_results:
            raise Exception(f"All servers failed to execute tool {tool_id}")
        
        # Simple case: only one successful result
        if len(successful_results) == 1:
            return list(successful_results.values())[0]
        
        # Tool-specific aggregation
        if tool_id.startswith("search."):
            return self._aggregate_search_results(successful_results)
        elif tool_id.startswith("weather."):
            return self._aggregate_weather_results(successful_results)
        elif tool_id.startswith("translate."):
            return self._aggregate_translation_results(successful_results)
        elif tool_id.startswith("analyze."):
            return self._aggregate_analysis_results(successful_results)
        else:
            # Default aggregation: return all results
            return {
                "aggregated": True,
                "servers": len(successful_results),
                "results": successful_results
            }
    
    def _aggregate_search_results(self, results: Dict[str, Any]) -> Any:
        """Aggregate search results"""
        all_items = []
        seen_items = set()
        
        # Collect all unique items
        for server_id, result in results.items():
            if "results" in result:
                items = result["results"]
                
                for item in items:
                    # Create a simple hash for deduplication
                    item_hash = self._hash_item(item)
                    
                    if item_hash not in seen_items:
                        seen_items.add(item_hash)
                        
                        # Add source information
                        item["_source"] = server_id
                        all_items.append(item)
        
        # Sort by relevance (assuming there's a relevance score)
        # In a real implementation, this would use a more sophisticated approach
        all_items.sort(key=lambda x: x.get("relevance", 0), reverse=True)
        
        return {
            "aggregated": True,
            "total": len(all_items),
            "results": all_items
        }
    
    def _aggregate_weather_results(self, results: Dict[str, Any]) -> Any:
        """Aggregate weather results"""
        # For weather, we'll take the average of numeric values
        aggregated = {}
        
        # Find common fields
        common_fields = set()
        for result in results.values():
            common_fields.update(result.keys())
        
        # Process each field
        for field in common_fields:
            values = []
            
            for result in results.values():
                if field in result and isinstance(result[field], (int, float)):
                    values.append(result[field])
            
            if values:
                # Calculate average for numeric fields
                aggregated[field] = sum(values) / len(values)
            else:
                # For non-numeric fields, use the most common value
                field_values = {}
                
                for result in results.values():
                    if field in result:
                        value = result[field]
                        
                        if isinstance(value, (list, dict)):
                            # Skip complex values for now
                            continue
                        
                        field_values[value] = field_values.get(value, 0) + 1
                
                if field_values:
                    # Get most common value
                    aggregated[field] = max(field_values.items(), key=lambda x: x[1])[0]
        
        # Add aggregation metadata
        aggregated["_aggregated"] = True
        aggregated["_sources"] = len(results)
        
        return aggregated
    
    def _aggregate_translation_results(self, results: Dict[str, Any]) -> Any:
        """Aggregate translation results"""
        # For translations, we'll use the most confident result
        best_result = None
        best_confidence = -1
        
        for server_id, result in results.items():
            confidence = result.get("confidence", 0)
            
            if confidence > best_confidence:
                best_confidence = confidence
                best_result = result
                best_result["_source"] = server_id
        
        if best_result:
            best_result["_aggregated"] = True
            best_result["_sources"] = len(results)
            
            return best_result
        else:
            # If no confidence scores, just return all results
            return {
                "_aggregated": True,
                "_sources": len(results),
                "translations": list(results.values())
            }
    
    def _aggregate_analysis_results(self, results: Dict[str, Any]) -> Any:
        """Aggregate analysis results"""
        # For analysis, we'll merge the results
        aggregated = {}
        
        # Collect all fields
        all_fields = set()
        for result in results.values():
            all_fields.update(result.keys())
        
        # Process each field
        for field in all_fields:
            field_values = []
            
            for result in results.values():
                if field in result:
                    value = result[field]
                    
                    if isinstance(value, list):
                        # For lists, collect all unique items
                        field_values.extend(value)
                    elif isinstance(value, dict):
                        # For dictionaries, we'd need a more complex merge
                        # For now, just take the first one
                        if not field in aggregated:
                            aggregated[field] = value
                    else:
                        # For scalars, collect all values
                        field_values.append(value)
            
            if field_values and field not in aggregated:
                # For scalars, either take the most common or average
                if all(isinstance(v, (int, float)) for v in field_values):
                    # Average for numeric values
                    aggregated[field] = sum(field_values) / len(field_values)
                else:
                    # Most common for non-numeric
                    value_counts = {}
                    
                    for value in field_values:
                        value_counts[value] = value_counts.get(value, 0) + 1
                    
                    most_common = max(value_counts.items(), key=lambda x: x[1])[0]
                    aggregated[field] = most_common
        
        # Add aggregation metadata
        aggregated["_aggregated"] = True
        aggregated["_sources"] = len(results)
        
        return aggregated
    
    def _hash_item(self, item):
        """Create a simple hash for an item for deduplication"""
        if isinstance(item, dict):
            # For dictionaries, hash based on important fields
            # This is a simplified example - real implementation would be more sophisticated
            important_fields = ["id", "title", "url", "content"]
            hash_parts = []
            
            for field in important_fields:
                if field in item:
                    hash_parts.append(str(item[field]))
            
            return "|".join(hash_parts)
        else:
            # For non-dictionaries, just use string representation
            return str(item)
```

## Practical Implementation Example

Here's a complete example of using the multi-server client with routing and aggregation:

```python
# Example usage of multi-server MCP client
import asyncio
import logging
from typing import Dict, List, Any

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

async def main():
    # Create multi-server client with routing and conflict resolution
    client = ConflictResolvingMultiServerMCPClient(
        routing_strategy=LatencyBasedRoutingStrategy(),
        conflict_resolver=CapabilityConflictResolver(
            default_strategy=ConflictResolutionStrategy.NEWEST_VERSION
        )
    )
    
    # Create result aggregator
    aggregator = ResultAggregator()
    
    # Add servers
    await client.add_server("server1", "http://localhost:8001", "token1")
    await client.add_server("server2", "http://localhost:8002", "token2")
    await client.add_server("server3", "http://localhost:8003", "token3")
    
    # Set server priorities
    client.conflict_resolver.set_server_priority("server1", 10)
    client.conflict_resolver.set_server_priority("server2", 5)
    client.conflict_resolver.set_server_priority("server3", 1)
    
    # Set user preferences
    client.conflict_resolver.set_user_preference("weather.forecast", "server2")
    
    # Set capability-specific conflict resolution strategies
    client.conflict_resolver.set_capability_strategy(
        "search.documents", 
        ConflictResolutionStrategy.MERGE
    )
    
    try:
        # Get all server status
        status = client.get_all_server_status()
        logger.info(f"Server status: {status}")
        
        # Invoke a tool on a specific server
        result1 = await client.invoke_tool("weather.current", {"location": "New York"}, "server1")
        logger.info(f"Weather result from server1: {result1}")
        
        # Invoke a tool with automatic routing
        result2 = await client.invoke_tool("search.documents", {"query": "multi-server architecture"})
        logger.info(f"Search result with routing: {result2}")
        
        # Get a resource with automatic server selection
        resource = await client.get_resource("documents.recent", {"count": 5})
        logger.info(f"Recent documents: {resource}")
        
        # Aggregate results from all servers
        aggregated_result = await aggregator.aggregate_tool_results(
            client, 
            "weather.forecast", 
            {"location": "London", "days": 3},
            strategy="all"
        )
        logger.info(f"Aggregated weather forecast: {aggregated_result}")
        
    finally:
        # Disconnect from all servers
        await client.disconnect_all_servers()

if __name__ == "__main__":
    asyncio.run(main())
```

## Benefits of Multi-Server Architecture

Using multiple MCP servers offers several advantages:

1. **Increased Capability Coverage**
   - Access to a wider range of tools and resources
   - Specialized servers for different domains
   - Redundancy for critical capabilities
   - Complementary functionality

2. **Enhanced Reliability**
   - Fault tolerance through redundancy
   - Graceful degradation on partial failures
   - High availability configurations
   - Load distribution

3. **Performance Optimization**
   - Load balancing across servers
   - Parallel request processing
   - Geographic distribution
   - Specialized hardware utilization

4. **Flexibility and Extensibility**
   - Easy addition of new capabilities
   - Dynamic server discovery
   - Progressive capability enhancement
   - Modular architecture

5. **Organizational Benefits**
   - Team-specific servers
   - Department-level specialization
   - Clear ownership boundaries
   - Simplified maintenance

## Challenges and Solutions

While multi-server architectures offer many benefits, they also introduce challenges:

| Challenge | Solution |
|-----------|----------|
| Capability conflicts | Implement conflict resolution strategies |
| Increased complexity | Use structured organization and clear naming conventions |
| Consistency issues | Implement synchronization mechanisms |
| Authentication management | Use federated authentication and SSO |
| Performance overhead | Optimize routing and connection management |
| Error handling | Implement comprehensive error handling and recovery |
| Resource consumption | Use connection pooling and resource optimization |
| Monitoring complexity | Implement aggregated monitoring and centralized logging |

## Design Patterns for Multi-Server Architecture

Several design patterns can help in implementing effective multi-server architectures:

1. **Service Registry Pattern**
   - Central registry for server discovery
   - Dynamic registration and deregistration
   - Health monitoring
   - Metadata management

2. **API Gateway Pattern**
   - Single entry point for clients
   - Request routing
   - Response aggregation
   - Cross-cutting concerns

3. **Circuit Breaker Pattern**
   - Fail fast for unavailable services
   - Automatic recovery
   - Graceful degradation
   - Monitoring and alerting

4. **Bulkhead Pattern**
   - Isolation of failures
   - Resource partitioning
   - Load shedding
   - Criticality-based prioritization

5. **Sidecar Pattern**
   - Auxiliary services for cross-cutting concerns
   - Authentication and authorization
   - Logging and monitoring
   - Rate limiting

## Summary

Managing connections to multiple MCP servers involves establishing connection management, handling server discovery, coordinating authentication, monitoring health, and implementing failover strategies. Routing requests across servers requires capability-based routing, load balancing, latency optimization, geographic distribution, and fallback mechanisms. Conflict resolution strategies address naming collisions, version management, capability prioritization, user preferences, and transparent handling. Aggregating results from multiple servers involves data fusion, result normalization, confidence scoring, conflict resolution, and presentation strategies.

By implementing these techniques, you can create robust multi-server MCP architectures that provide increased capability coverage, enhanced reliability, performance optimization, flexibility, and organizational benefits.
