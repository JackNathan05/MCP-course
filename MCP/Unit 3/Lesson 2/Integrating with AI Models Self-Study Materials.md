
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 3/Lesson 2/_Lesson 2|_Lesson 2]]

---

## Practical Exercise 3.2

**Objective**: Integrate an MCP client with an AI model

### Instructions

1. Set up access to an AI model with function calling capabilities:
   - OpenAI API (GPT-4)
   - Anthropic API (Claude)
   - Or another model with similar capabilities
   - Configure API keys and client libraries

2. Connect your MCP client from Exercise 3.1:
   - Ensure your client can connect to your MCP server
   - Verify it can discover capabilities
   - Test tool invocation

3. Translate MCP tool definitions to function schemas:
   - Create conversion functions for your chosen AI model
   - Preserve parameter details and descriptions
   - Include validation constraints
   - Test schema conversion

4. Implement the full interaction flow:
   - Send user query to AI model with available functions
   - Parse model response for function calls
   - Execute requested tools via MCP
   - Return results to the model
   - Present final response to the user

### Expected Outcome

A working system that:
- Takes natural language queries from users
- Connects to both an AI model and MCP server
- Allows the AI model to invoke MCP tools
- Returns tool results back to the model
- Presents a coherent response to the user

## Code Example (JavaScript)

Here's a starter implementation for integrating MCP with OpenAI:

```javascript
// Example integration with OpenAI model
import { MCPClient } from 'mcp-client';
import { OpenAI } from 'openai';

// Initialize clients
const mcpClient = new MCPClient();
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Connect to MCP server
async function connectToMCPServer() {
  try {
    await mcpClient.connect('http://localhost:8000');
    console.log('Connected to MCP server');
    
    // Get capabilities
    const capabilities = await mcpClient.getCapabilities();
    console.log(`Discovered ${capabilities.length} capabilities`);
    
    return capabilities;
  } catch (error) {
    console.error('Failed to connect to MCP server:', error);
    throw error;
  }
}

// Convert MCP tools to OpenAI function definitions
function convertToolsToFunctions(tools) {
  return tools
    .filter(tool => tool.type === 'tool')
    .map(tool => {
      // Convert tool ID to function name (snake_case)
      const functionName = tool.id.replace(/\./g, '_');
      
      // Create properties object for parameters
      const properties = {};
      const required = [];
      
      tool.parameters.forEach(param => {
        properties[param.name] = {
          type: param.type,
          description: param.description
        };
        
        // Add additional constraints
        if (param.enum) properties[param.name].enum = param.enum;
        if (param.minimum !== undefined) properties[param.name].minimum = param.minimum;
        if (param.maximum !== undefined) properties[param.name].maximum = param.maximum;
        if (param.default !== undefined) properties[param.name].default = param.default;
        
        if (param.required) required.push(param.name);
      });
      
      return {
        name: functionName,
        description: tool.description,
        parameters: {
          type: 'object',
          properties,
          required
        }
      };
    });
}

// Process user query
async function processUserQuery(userQuery, conversationHistory = []) {
  try {
    // Connect to MCP server and get capabilities
    const capabilities = await connectToMCPServer();
    
    // Convert tools to function definitions
    const functionDefinitions = convertToolsToFunctions(capabilities);
    
    // Prepare messages for OpenAI
    const messages = [
      { role: 'system', content: 'You are a helpful assistant with access to tools. Use tools when necessary to answer the user\'s question.' },
      ...conversationHistory,
      { role: 'user', content: userQuery }
    ];
    
    // Call OpenAI API with function definitions
    const response = await openai.chat.completions.create({
      model: 'gpt-4',
      messages,
      functions: functionDefinitions,
      function_call: 'auto'
    });
    
    // Get the assistant's message
    const assistantMessage = response.choices[0].message;
    
    // Check if the model wants to call a function
    if (assistantMessage.function_call) {
      // Extract function details
      const { name, arguments: args } = assistantMessage.function_call;
      
      console.log(`Model wants to call function: ${name}`);
      console.log(`Arguments: ${args}`);
      
      // Convert function name back to MCP tool ID
      const toolId = name.replace(/_/g, '.');
      
      // Parse arguments
      const parsedArgs = JSON.parse(args);
      
      // Call the MCP tool
      const result = await mcpClient.invokeTool(toolId, parsedArgs);
      
      // Add the function call and result to messages
      messages.push({
        role: 'assistant',
        content: null,
        function_call: {
          name,
          arguments: args
        }
      });
      
      messages.push({
        role: 'function',
        name,
        content: JSON.stringify(result)
      });
      
      // Call OpenAI again to process the function result
      const finalResponse = await openai.chat.completions.create({
        model: 'gpt-4',
        messages
      });
      
      return finalResponse.choices[0].message.content;
    } else {
      // Model didn't call a function, return the response directly
      return assistantMessage.content;
    }
  } catch (error) {
    console.error('Error processing query:', error);
    return `Error: ${error.message}`;
  } finally {
    // Disconnect from MCP server
    await mcpClient.disconnect();
  }
}

// Example usage
async function main() {
  const userQuery = "What's the weather like in San Francisco today?";
  const response = await processUserQuery(userQuery);
  console.log('AI Response:', response);
}

main().catch(console.error);
```

## Extension Activities

1. **Multi-turn Conversation**:
   - Implement conversation history management
   - Add context tracking across turns
   - Handle reference resolution (e.g., "it", "that", etc.)
   - Create a simple chat interface

2. **Advanced Tool Selection Logic**:
   - Implement tool selection guidance
   - Add tool categorization
   - Create usage examples for each tool
   - Test with ambiguous queries

3. **Performance Optimization**:
   - Implement connection pooling
   - Add caching for capabilities and schemas
   - Optimize context management
   - Measure and improve response times

## Recommended Reading

1. [MCP Model Integration Guide](https://modelcontextprotocol.io/docs/guides/model-integration/)
   - Best practices for model integration
   - Context management strategies
   - Response handling patterns

2. [Function Calling Best Practices](https://modelcontextprotocol.io/docs/guides/function-calling/)
   - Schema design recommendations
   - Parameter validation
   - Error handling
   - Model-specific considerations

3. [OpenAI Function Calling API](https://platform.openai.com/docs/guides/function-calling)
   - Official documentation
   - Implementation examples
   - Schema requirements
   - Best practices

4. [Anthropic Claude API](https://docs.anthropic.com/claude/reference/tools)
   - Tool use documentation
   - Schema format
   - Response handling
   - Limitations and guidelines

## Self-Assessment Questions

Test your understanding by answering these questions:

1. How should MCP tool definitions be translated to model function schemas?
2. What context information should be provided to AI models?
3. How should function call responses be processed?
4. What strategies can help AI models select the most appropriate tools?
5. How should errors in tool execution be communicated back to the AI model?
6. What are the key differences between OpenAI and Anthropic function calling formats?
7. How can you optimize context management for large tool sets?
8. What security considerations are important when allowing AI models to invoke tools?

## Schema Conversion Exercise

Create conversion functions for translating MCP tool definitions to function schemas for:

1. **OpenAI**:
   - Review OpenAI's function calling format
   - Implement a conversion function
   - Test with various tool types
   - Handle edge cases (e.g., nested objects, arrays)

2. **Anthropic Claude**:
   - Review Claude's tool use format
   - Implement a conversion function
   - Test with various tool types
   - Handle Claude-specific requirements

3. **A Third Model of Your Choice**:
   - Find documentation for another model with function calling
   - Identify format differences
   - Implement a conversion function
   - Test with sample tools

Compare the differences between the formats and discuss the implications for MCP integration.

## Context Management Implementation

Design and implement an effective context management system:

1. Create a context manager class that:
   - Tracks conversation history
   - Manages resource data
   - Handles context prioritization
   - Respects model context limits

2. Implement context compression strategies:
   - Summarization of conversation history
   - Selective inclusion of resources
   - Priority-based truncation
   - Contextual relevance scoring

3. Test with different context scenarios:
   - Short conversations with minimal context
   - Long conversations with extensive history
   - Resource-heavy interactions
   - Mixed context types

Document your findings on context management efficiency and model performance.

## Additional Resources

### AI Model Integration
- [OpenAI Function Calling Examples](https://github.com/openai/openai-cookbook/tree/main/examples/function_calling)
- [Claude Tool Use Guide](https://docs.anthropic.com/claude/docs/tool-use)
- [Tool-Augmented LLMs](https://modelcontextprotocol.io/docs/guides/tool-augmented-llms/)

### Schema Design
- [JSON Schema Best Practices](https://modelcontextprotocol.io/docs/guides/json-schema/)
- [Parameter Design Patterns](https://modelcontextprotocol.io/docs/patterns/parameters/)
- [Tool Description Guidelines](https://modelcontextprotocol.io/docs/guides/tool-descriptions/)

### Security and Performance
- [AI Safety with Tool Use](https://modelcontextprotocol.io/docs/guides/ai-safety/)
- [Performance Optimization for Model Integration](https://modelcontextprotocol.io/docs/guides/model-performance/)
- [Security Best Practices for LLM Tools](https://modelcontextprotocol.io/docs/guides/llm-tool-security/)
