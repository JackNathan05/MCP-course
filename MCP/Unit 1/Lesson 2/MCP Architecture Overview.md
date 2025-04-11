
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 1/Lesson 2/_Lesson 2]]

---

## Overview

In this lesson, we'll explore the core architecture of the Model Context Protocol, understanding its components and how they interact with each other.

## Client-Server Model Fundamentals

MCP follows a client-server architecture with specific components:

The MCP ecosystem consists of several key components:

1. **MCP Hosts**: Applications like Claude Desktop or AI-driven IDEs that need access to external data or tools
2. **MCP Clients**: Components that maintain dedicated connections with MCP servers
3. **MCP Servers**: Lightweight servers exposing specific functionalities via MCP, connecting to data sources
4. **Local Data Sources**: Files, databases, or services securely accessed by MCP servers
5. **Remote Services**: External internet-based APIs or services accessed by MCP servers

This architecture creates a clean separation of concerns where:
- Hosts focus on user interaction and model integration
- Clients handle communication with servers
- Servers provide access to data and functionality

## Key Components and Concepts

MCP defines three primary types of capabilities:

1. **Tools (Model-controlled)**: Actions the AI decides to take, similar to function calling. These are methods that can be invoked to perform specific tasks, like sending an email, retrieving weather data, or analyzing code.

2. **Resources (Application-controlled)**: Data sources that LLMs can access, similar to GET endpoints in a REST API, providing data without significant computation or side effects. These are typically part of the context/request to the AI.

3. **Prompts (User-controlled)**: Pre-defined templates to use tools or resources in the most optimal way. These are selected before running inference.

Each capability has standardized metadata including:
- Unique identifier
- Human-readable description
- Input/output schema
- Authentication requirements
- Optional metadata like version, category, etc.

## Communication Flow in MCP

MCP operates on a structured communication flow:

1. **Initialization**: When a Host application starts, it creates MCP Clients, which exchange capabilities and protocol versions via a handshake

2. **Discovery**: Clients request what capabilities the server offers, and the Server responds with descriptions

3. **Context Provision**: The Host application makes resources available to the user or parses tools into an LLM-compatible format

4. **Invocation**: If the LLM needs to use a Tool, the Host directs the Client to send an invocation request

5. **Execution**: The Server executes the underlying logic and returns the result

This flow ensures that AI models only have access to capabilities explicitly provided by MCP servers, and that all interactions follow a consistent pattern.

## Protocol Versioning and Compatibility

MCP is under active development, with regular version updates to improve security, functionality, and usability. The specification follows semantic versioning, and includes clear compatibility guidelines.

The latest version (as of 2025-03-26) includes improvements in areas like:
- **Authentication & Security**: Mandating OAuth 2.1 for authenticating remote HTTP servers
- **Transport & Efficiency**: More flexible transport options and support for batching
- **Context & Control**: Enhanced metadata about tool behaviors

When implementing MCP, it's important to:
- Specify which version of the protocol you're using
- Handle version negotiation during connection
- Consider backward compatibility for clients and servers

## Transport Mechanisms

MCP supports multiple transport mechanisms for flexibility:

1. **HTTP with Server-Sent Events (SSE)**: Most common for web applications
2. **WebSockets**: For real-time bidirectional communication
3. **STDIO**: For local process communication
4. **UNIX sockets**: For inter-process communication on Unix-like systems

The choice of transport depends on the specific needs of your application, including:
- Performance requirements
- Security considerations
- Deployment environment
- Integration with existing systems

## Diagram: MCP Architecture

```
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│               │      │               │      │               │
│  MCP Host     │      │  MCP Server 1 │      │ Data Source 1 │
│ (Application) │      │ (e.g., GitHub)│◄────►│ (GitHub API)  │
│               │      │               │      │               │
└───────┬───────┘      └───────────────┘      └───────────────┘
        │
        │ Creates
        ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│               │      │               │      │               │
│  MCP Client 1 ├─────►│  MCP Server 2 │◄────►│ Data Source 2 │
│               │      │ (e.g., Files) │      │ (Local Files) │
└───────┬───────┘      └───────────────┘      └───────────────┘
        │
        │ Also Creates
        ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│               │      │               │      │               │
│  MCP Client 2 ├─────►│  MCP Server 3 │◄────►│ Data Source 3 │
│               │      │ (e.g., DB)    │      │ (Database)    │
└───────────────┘      └───────────────┘      └───────────────┘
```

## Security Considerations

Security is a critical aspect of the MCP architecture:

- Servers should validate client authentication and authorization
- Tools require explicit user consent before invocation
- Resources should implement appropriate access controls
- Communication should be encrypted
- Sensitive operations should be logged for audit purposes

The protocol itself doesn't enforce security, but provides recommendations and standards that implementations should follow.

## Summary

The MCP architecture provides a flexible, standardized way for AI models to interact with external systems. By understanding the client-server model, communication flow, and core components, you'll be able to design and implement effective MCP integrations.
