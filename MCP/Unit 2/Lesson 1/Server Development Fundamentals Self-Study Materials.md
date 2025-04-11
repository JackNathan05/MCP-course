
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 2/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Practical Exercise 2.1

**Objective**: Create a basic MCP server

### Instructions

1. Choose a programming language and install the appropriate MCP SDK:
   - Python: `pip install mcp-server`
   - Node.js: `npm install @modelcontextprotocol/server`
   - Java: Add the Maven dependency to your `pom.xml`
   - C#: `dotnet add package ModelContextProtocol.Server`

2. Create a simple MCP server that exposes one tool capability:
   - Choose a simple function like a calculator, weather lookup, or text processor
   - Define appropriate parameters and return types
   - Write a clear description of the tool's functionality

3. Implement the discovery endpoints:
   - Ensure the server properly advertises its capabilities
   - Test that it correctly responds to discovery requests

4. Test the server using an MCP client or testing tool:
   - Use the MCP Inspector tool or a simple client
   - Verify that the tool can be discovered and invoked
   - Test both successful invocation and error handling

### Expected Outcome

A working MCP server that:
- Starts up and listens for connections
- Responds to capability discovery requests
- Successfully handles tool invocations
- Returns appropriate responses and errors

## Code Example (Python)

Here's a starter implementation for a basic calculator server in Python:

```python
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter

# Create server
server = MCPServer()

# Define calculator tool
def calculator(operation: str, a: float, b: float) -> float:
    """A simple calculator tool"""
    if operation == "add":
        return a + b
    elif operation == "subtract":
        return a - b
    elif operation == "multiply":
        return a * b
    elif operation == "divide":
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b
    else:
        raise ValueError(f"Unknown operation: {operation}")

# Register tool
server.register_tool(
    id="calculator",
    function=calculator,
    description="A simple calculator that performs basic operations",
    parameters=[
        MCPParameter(
            name="operation",
            description="The operation to perform (add, subtract, multiply, divide)",
            type="string",
            enum=["add", "subtract", "multiply", "divide"],
            required=True
        ),
        MCPParameter(
            name="a",
            description="The first number",
            type="number",
            required=True
        ),
        MCPParameter(
            name="b",
            description="The second number",
            type="number",
            required=True
        )
    ],
    return_type="number"
)

# Start server
if __name__ == "__main__":
    server.start(port=8000)
    print("Server started on port 8000")
```

## Extension Activities

1. **Add Error Handling**:
   - Implement robust error handling for edge cases
   - Return descriptive error messages
   - Log errors for debugging

2. **Add Authentication**:
   - Implement basic authentication (API key or simple token)
   - Verify authentication on tool invocation
   - Reject unauthorized requests

3. **Add a Resource**:
   - Implement a simple resource provider
   - Make it return dynamic data based on parameters
   - Ensure proper access control

## Recommended Reading

1. [MCP Server Implementation Guide](https://modelcontextprotocol.io/docs/guides/server-implementation/)
   - Best practices for server implementation
   - Configuration options
   - Performance considerations

2. [MCP SDK Documentation](https://github.com/modelcontextprotocol/)
   - Language-specific SDK documentation
   - API references
   - Example implementations

3. [Error Handling in MCP](https://modelcontextprotocol.io/docs/guides/error-handling/)
   - Standard error formats
   - Error codes and categories
   - Best practices for error messages

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What are the main components needed to set up an MCP server?
2. How does the capability discovery process work?
3. What information must be included in a tool declaration?
4. How should an MCP server handle errors?
5. What transport mechanisms can an MCP server support?
6. What security considerations are important when implementing an MCP server?
7. How can you test an MCP server without building a client?
8. What are the recommended practices for structuring tool IDs and names?

## Debugging Exercise

Review this problematic MCP server code and identify the issues:

```python
from mcp.server import MCPServer

server = MCPServer()

# This tool has several issues
server.register_tool(
    id="search",  # Missing namespace
    # Missing description
    parameters=[
        # Parameters not properly defined
        {"name": "query"}
    ],
    # Return type not specified
    handler=lambda params: {"results": ["result1", "result2"]}
)

# Missing error handling
server.start()
```

Identify at least 5 issues with this code and explain how to fix them.

## Additional Resources

### Testing Tools
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector) - Interactive tool for testing MCP servers
- [MCP CLI](https://github.com/modelcontextprotocol/cli) - Command-line tool for interacting with MCP servers

### Sample Implementations
- [Reference MCP Server](https://github.com/modelcontextprotocol/examples/server) - Example server implementations
- [MCP Server Templates](https://github.com/modelcontextprotocol/templates) - Starter templates for different languages

### Deployment Guides
- [Docker Deployment](https://modelcontextprotocol.io/docs/guides/deployment/docker)
- [Cloud Deployment](https://modelcontextprotocol.io/docs/guides/deployment/cloud)
