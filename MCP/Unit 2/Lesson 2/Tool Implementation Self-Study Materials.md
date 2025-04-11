
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 2/Lesson 2/_Lesson 2|_Lesson 2]]

---

## Practical Exercise 2.2

**Objective**: Implement multiple tools in an MCP server

### Instructions

1. Extend your MCP server from Exercise 2.1 with three new tools of different complexity:

   a. **Calculator Tool**
   - Implement a calculator that performs basic math operations
   - Support addition, subtraction, multiplication, division
   - Include proper error handling for division by zero and invalid operations
   - Provide clear parameter descriptions for AI models to use effectively

   b. **Weather Tool**
   - Create a tool that returns weather information for a given location
   - You can either integrate with a real weather API or mock the response
   - Include parameters for location and optional units (celsius/fahrenheit)
   - Add appropriate error handling for invalid locations

   c. **Text Analysis Tool**
   - Build a tool that analyzes text to provide statistics
   - Count words, characters, sentences, and paragraphs
   - Detect sentiment (positive, negative, neutral) if possible
   - Handle formatting and return structured results

2. Ensure each tool has:
   - Proper documentation and descriptive metadata
   - Well-defined input/output schemas
   - Comprehensive error handling
   - Appropriate logging

3. Test each tool with both valid and invalid inputs:
   - Create test cases for normal operation
   - Test edge cases (empty inputs, very large inputs)
   - Test error scenarios (invalid parameters, service unavailability)
   - Document your test results

### Expected Outcome

A working MCP server with three tools that:
- Are properly documented
- Have well-defined schemas
- Handle errors effectively
- Produce consistent results

## Code Example (TypeScript)

Here's a starter implementation for the calculator tool:

```typescript
// calculator-tool.ts
import { MCPTool } from 'mcp-server';

export const calculatorTool: MCPTool = {
  id: 'math.calculator',
  name: 'Calculator',
  description: 'Performs basic math operations',
  parameters: [
    {
      name: 'operation',
      description: 'Math operation to perform',
      type: 'string',
      enum: ['add', 'subtract', 'multiply', 'divide'],
      required: true
    },
    {
      name: 'a',
      description: 'First number',
      type: 'number',
      required: true
    },
    {
      name: 'b',
      description: 'Second number',
      type: 'number',
      required: true
    }
  ],
  handler: async (params) => {
    const { operation, a, b } = params;
    
    // Validate input types
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Both a and b must be numbers');
    }
    
    // Perform calculation based on operation
    switch (operation) {
      case 'add':
        return { result: a + b };
      case 'subtract':
        return { result: a - b };
      case 'multiply':
        return { result: a * b };
      case 'divide':
        if (b === 0) {
          throw new Error('Division by zero');
        }
        return { result: a / b };
      default:
        throw new Error('Unknown operation');
    }
  }
};

// Register in main server file
import { MCPServer } from 'mcp-server';
import { calculatorTool } from './calculator-tool';

const server = new MCPServer();
server.registerTool(calculatorTool);
```

Here's a starter implementation for the text analysis tool:

```python
# text_analysis_tool.py
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter
import re

def analyze_text(text: str):
    """Analyze text and return statistics"""
    # Validate input
    if not text or not isinstance(text, str):
        raise ValueError("Input must be a non-empty string")
    
    # Count characters
    char_count = len(text)
    char_count_no_spaces = len(text.replace(" ", ""))
    
    # Count words
    words = re.findall(r'\b\w+\b', text)
    word_count = len(words)
    
    # Count sentences
    sentences = re.split(r'[.!?]+', text)
    sentence_count = len([s for s in sentences if s.strip()])
    
    # Count paragraphs
    paragraphs = text.split('\n\n')
    paragraph_count = len([p for p in paragraphs if p.strip()])
    
    # Simple sentiment analysis
    positive_words = ['good', 'great', 'excellent', 'happy', 'positive', 'wonderful', 'amazing']
    negative_words = ['bad', 'terrible', 'awful', 'sad', 'negative', 'horrible', 'disappointing']
    
    lowercase_text = text.lower()
    positive_count = sum(1 for word in positive_words if word in lowercase_text)
    negative_count = sum(1 for word in negative_words if word in lowercase_text)
    
    if positive_count > negative_count:
        sentiment = "positive"
    elif negative_count > positive_count:
        sentiment = "negative"
    else:
        sentiment = "neutral"
    
    # Return analysis results
    return {
        "statistics": {
            "characters": char_count,
            "characters_no_spaces": char_count_no_spaces,
            "words": word_count,
            "sentences": sentence_count,
            "paragraphs": paragraph_count
        },
        "sentiment": sentiment
    }

# Create server
server = MCPServer()

# Register tool
server.register_tool(
    id="text.analyze",
    function=analyze_text,
    description="Analyzes text and provides statistics and basic sentiment analysis",
    parameters=[
        MCPParameter(
            name="text",
            description="The text to analyze",
            type="string",
            required=True
        )
    ],
    return_schema={
        "type": "object",
        "properties": {
            "statistics": {
                "type": "object",
                "properties": {
                    "characters": {"type": "integer"},
                    "characters_no_spaces": {"type": "integer"},
                    "words": {"type": "integer"},
                    "sentences": {"type": "integer"},
                    "paragraphs": {"type": "integer"}
                }
            },
            "sentiment": {
                "type": "string",
                "enum": ["positive", "negative", "neutral"]
            }
        }
    }
)
```

## Extension Activities

1. **Tool Integration**:
   - Connect your tools to external services (weather API, database, etc.)
   - Implement caching for expensive operations
   - Add rate limiting to prevent abuse

2. **Advanced Tool Features**:
   - Add support for asynchronous operations
   - Implement progress reporting for long-running tasks
   - Add support for streaming responses

3. **Security Enhancements**:
   - Add input validation and sanitization
   - Implement access control based on user roles
   - Add audit logging for sensitive operations

## Recommended Reading

1. [MCP Tool Implementation Best Practices](https://modelcontextprotocol.io/docs/guides/tool-implementation/)
   - Guidelines for designing effective tools
   - Error handling patterns
   - Performance optimization techniques

2. [Schema Design Guidelines](https://modelcontextprotocol.io/docs/guides/schema-design/)
   - Best practices for parameter schemas
   - Type system details
   - Validation approaches

3. [Tool Security Considerations](https://modelcontextprotocol.io/docs/guides/tool-security/)
   - Input validation techniques
   - Authorization patterns
   - Secure integration with external services

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What key elements should be included in a tool definition?
2. How should tool parameters be structured for best usability?
3. What are some best practices for error handling in tool implementations?
4. How can you ensure tools are secure and prevent misuse?
5. What considerations should be made when designing tool response formats?
6. How can you make tools more discoverable and useful for AI models?
7. What are the different types of errors a tool might encounter and how should they be handled?
8. How would you design a tool to work with sensitive data?

## Tool Design Workshop

Design documentation for a new MCP tool according to these specifications:

1. Choose one of these domains:
   - A financial tool (e.g., currency conversion, stock price checking)
   - A productivity tool (e.g., task creation, calendar management)
   - A content generation tool (e.g., image description, code formatting)

2. Create comprehensive documentation including:
   - Tool ID and name
   - Detailed description
   - Complete parameter schema
   - Return value schema
   - Example invocations and responses
   - Error scenarios and responses
   - Security considerations

3. Review your design considering:
   - Is it easy for an AI model to understand when to use this tool?
   - Are the parameters clearly defined and validated?
   - Does the return format provide useful information?
   - Have you handled potential error cases?

## Additional Resources

### Tool Design Patterns
- [Common Tool Patterns](https://modelcontextprotocol.io/docs/patterns/tools/)
- [Parameter Design Best Practices](https://modelcontextprotocol.io/docs/patterns/parameters/)

### Example Tools
- [MCP Tool Gallery](https://modelcontextprotocol.io/tools/)
- [Community Tool Repository](https://github.com/modelcontextprotocol/community-tools)

### Testing Frameworks
- [MCP Tool Testing](https://modelcontextprotocol.io/docs/guides/tool-testing/)
- [Schema Validation](https://modelcontextprotocol.io/docs/guides/schema-validation/)
