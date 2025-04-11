
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 1/Lesson 3/_Lesson 3]]

---

## Practical Exercise 1.3

**Objective**: Understand MCP message formats

### Instructions

1. Review the MCP specification documentation for message formats
2. Create example JSON messages for:
   - A tool discovery request
   - A tool invocation request
   - A tool invocation response
   - An error response
3. Identify which fields are required versus optional in each message type
4. Create a schema validator that could check if messages conform to the MCP specification

### Expected Outcome

A document containing:
- Example JSON messages for each message type
- Tables showing required and optional fields
- A simple schema validation function (pseudocode or actual code)

## Example Messages

Here's a starting point for your tool invocation request:

```json
{
  "jsonrpc": "2.0",
  "id": "request-123",
  "method": "tool/invoke",
  "params": {
    "tool_id": "search_documents",
    "parameters": {
      "query": "artificial intelligence",
      "max_results": 10,
      "filter": {
        "date_range": {
          "start": "2024-01-01",
          "end": "2025-04-10"
        }
      }
    }
  }
}
```

## Recommended Reading

1. [MCP Specification - Protocol Details](https://spec.modelcontextprotocol.io/specification/)
   - Focus on the sections covering message formats
   - Pay special attention to the error handling standards

2. [Security Best Practices for MCP](https://modelcontextprotocol.io/introduction)
   - Authentication mechanisms
   - Authorization patterns
   - Data protection strategies

3. [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
   - This forms the foundation of MCP's messaging format
   - Understanding this will help you grasp MCP's extensions

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What transport mechanisms does MCP support?
2. What messaging format does MCP use as its foundation?
3. What are the required fields in a tool invocation request?
4. What authentication framework does MCP recommend for remote HTTP servers?
5. What security considerations should MCP implementers address?
6. How are errors communicated in MCP?
7. What is the difference between a protocol error and an application error in MCP?
8. How does MCP handle streaming responses?

## Hands-On Implementation Exercise

Create a simple schema validator for MCP messages using a language of your choice. Here's a pseudocode example to get you started:

```python
def validate_tool_invocation_request(message):
    # Check required fields
    if "jsonrpc" not in message or message["jsonrpc"] != "2.0":
        return False, "Missing or invalid jsonrpc version"
    
    if "id" not in message:
        return False, "Missing request id"
    
    if "method" not in message or message["method"] != "tool/invoke":
        return False, "Missing or invalid method"
    
    if "params" not in message:
        return False, "Missing params object"
    
    params = message["params"]
    
    if "tool_id" not in params:
        return False, "Missing tool_id in params"
    
    if "parameters" not in params:
        return False, "Missing parameters in params"
    
    # Message is valid
    return True, "Valid tool invocation request"
```

Extend this to create validators for other message types.

## Security Analysis Exercise

Review the security considerations in the MCP specification and create a security checklist for MCP implementers. Include:

1. Authentication requirements
2. Authorization patterns
3. Input validation approaches
4. Output sanitization methods
5. Rate limiting strategies
6. Audit logging recommendations

## Additional Resources

### Transport Mechanism Comparison
- [HTTP+SSE vs WebSocket Performance](https://modelcontextprotocol.io/docs/transport/comparison)
- [Implementing STDIO Transport](https://modelcontextprotocol.io/docs/transport/stdio)

### Error Handling
- [MCP Error Codes Reference](https://modelcontextprotocol.io/docs/errors/codes)
- [Best Practices for Error Handling](https://modelcontextprotocol.io/docs/errors/practices)

### Security
- [OAuth 2.1 for MCP](https://modelcontextprotocol.io/docs/security/oauth)
- [MCP Security Checklist](https://modelcontextprotocol.io/docs/security/checklist)
