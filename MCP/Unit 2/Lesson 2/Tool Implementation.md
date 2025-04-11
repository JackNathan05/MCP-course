
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 2/Lesson 2/_Lesson 2|_Lesson 2]]

---

## Overview

This lesson focuses on creating and exposing tools through MCP, including tool definitions, schemas, and implementation.

## Creating and Exposing Tools via MCP

Tools are functions that an AI model can invoke to perform actions. Implementing tools involves:

1. **Defining the tool's functionality**
   - What action will the tool perform?
   - What inputs does it need?
   - What outputs will it produce?
   - What side effects might it have?

2. **Creating clear descriptions and documentation**
   - Purpose and use cases
   - Parameter explanations
   - Return value descriptions
   - Limitations and constraints

3. **Specifying input/output schemas**
   - Parameter types and formats
   - Required vs. optional parameters
   - Default values
   - Validation rules

4. **Implementing business logic**
   - Core functionality
   - External service integration
   - Error handling
   - Performance considerations

5. **Handling errors and edge cases**
   - Input validation
   - Service availability issues
   - Resource constraints
   - Security concerns

## Tool Definitions and Descriptions

Tool definitions must include:

- **Unique identifier**
  - Should be descriptive and namespaced
  - Follow a consistent naming convention
  - Avoid conflicts with other tools

- **Human-readable name and description**
  - Clear, concise titles
  - Detailed descriptions
  - Appropriate for both AI models and humans

- **Purpose and use cases**
  - When should the tool be used?
  - What problems does it solve?
  - Example scenarios

- **Input parameters with types and descriptions**
  - Parameter names and types
  - Clear descriptions
  - Validation constraints

- **Output format and example responses**
  - Return type and structure
  - Example successful responses
  - Example error responses

- **Error scenarios**
  - Common error cases
  - How errors are communicated
  - How to handle errors

Good tool descriptions help AI models understand when and how to use the tool. They should be clear, detailed, and provide enough context for the model to make appropriate decisions.

## Input/Output Schema Design

Well-designed schemas are critical for tool usability:

- **Clear parameter naming conventions**
  - Descriptive names
  - Consistent style (camelCase, snake_case)
  - Avoid abbreviations unless standard

- **Appropriate data types**
  - String, number, boolean, object, array
  - Complex types when needed
  - Constraints (min/max values, patterns)

- **Validation rules**
  - Required fields
  - Format validation (email, URL, etc.)
  - Range constraints
  - Enum values

- **Required vs. optional parameters**
  - Mark truly required parameters
  - Provide sensible defaults for optional ones
  - Document default behaviors

- **Structured response formats**
  - Consistent structure
  - Clear field names
  - Appropriate types
  - Handle empty/null values

## Implementing Tool Logic

The actual implementation should:

- **Follow the defined schema**
  - Validate inputs against schema
  - Return outputs matching schema
  - Handle type conversions

- **Include proper error handling**
  - Input validation
  - External service errors
  - Internal errors
  - Timeouts

- **Maintain security and access controls**
  - Authenticate requests
  - Authorize operations
  - Validate inputs
  - Sanitize outputs

- **Be performant and reliable**
  - Optimize costly operations
  - Implement caching when appropriate
  - Handle concurrent requests
  - Respect timeouts

- **Include logging and monitoring**
  - Request/response logging
  - Error logging
  - Performance metrics
  - Usage statistics

## Tool Implementation Examples

Let's look at some example tool implementations in different languages:

### Python Example: Weather Forecast Tool

```python
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter
import requests
import os

# Create server
server = MCPServer()

# Weather API key
WEATHER_API_KEY = os.environ.get("WEATHER_API_KEY")

def get_weather(location: str, days: int = 3):
    """Get weather forecast for a location"""
    try:
        # Input validation
        if not location or not isinstance(location, str):
            raise ValueError("Location must be a non-empty string")
        
        if not isinstance(days, int) or days < 1 or days > 10:
            raise ValueError("Days must be an integer between 1 and 10")
        
        # Call external weather API
        response = requests.get(
            "https://api.weatherservice.com/forecast",
            params={
                "location": location,
                "days": days,
                "apikey": WEATHER_API_KEY
            },
            timeout=5  # Set timeout
        )
        
        # Check for errors
        response.raise_for_status()
        
        # Parse response
        data = response.json()
        
        # Format and return results
        return {
            "location": data["location"]["name"],
            "country": data["location"]["country"],
            "forecast": [
                {
                    "date": day["date"],
                    "condition": day["condition"]["text"],
                    "max_temp_c": day["temp_max_c"],
                    "min_temp_c": day["temp_min_c"],
                    "chance_of_rain": day["chance_of_rain"]
                }
                for day in data["forecast"]["forecastday"][:days]
            ]
        }
        
    except requests.exceptions.RequestException as e:
        # Handle API errors
        raise RuntimeError(f"Weather service error: {str(e)}")
    except (KeyError, ValueError, TypeError) as e:
        # Handle data parsing errors
        raise ValueError(f"Error processing weather data: {str(e)}")

# Register tool
server.register_tool(
    id="weather.forecast",
    function=get_weather,
    description="Get weather forecast for a specific location",
    parameters=[
        MCPParameter(
            name="location",
            description="City name or geographic coordinates",
            type="string",
            required=True
        ),
        MCPParameter(
            name="days",
            description="Number of days to forecast (1-10)",
            type="integer",
            minimum=1,
            maximum=10,
            default=3,
            required=False
        )
    ],
    return_schema={
        "type": "object",
        "properties": {
            "location": {"type": "string"},
            "country": {"type": "string"},
            "forecast": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "date": {"type": "string"},
                        "condition": {"type": "string"},
                        "max_temp_c": {"type": "number"},
                        "min_temp_c": {"type": "number"},
                        "chance_of_rain": {"type": "number"}
                    }
                }
            }
        }
    }
)

# Start server
if __name__ == "__main__":
    server.start(port=8000)
```

### TypeScript Example: Document Search Tool

```typescript
import { MCPServer, MCPTool } from 'mcp-server';
import { searchDocuments } from './document-service';

// Create server
const server = new MCPServer();

// Register document search tool
const searchTool: MCPTool = {
  id: 'document.search',
  name: 'Document Search',
  description: 'Search for documents using keywords and filters',
  parameters: [
    {
      name: 'query',
      description: 'Search keywords',
      type: 'string',
      required: true
    },
    {
      name: 'max_results',
      description: 'Maximum number of results to return',
      type: 'integer',
      minimum: 1,
      maximum: 50,
      default: 10,
      required: false
    },
    {
      name: 'date_range',
      description: 'Date range filter',
      type: 'object',
      required: false,
      properties: {
        start: {
          type: 'string',
          format: 'date',
          description: 'Start date (YYYY-MM-DD)'
        },
        end: {
          type: 'string',
          format: 'date',
          description: 'End date (YYYY-MM-DD)'
        }
      }
    },
    {
      name: 'document_type',
      description: 'Filter by document type',
      type: 'string',
      enum: ['pdf', 'docx', 'txt', 'markdown', 'all'],
      default: 'all',
      required: false
    }
  ],
  handler: async (params, context) => {
    try {
      // Get user from context for authorization
      const user = context.user;
      
      // Input validation
      if (!params.query || typeof params.query !== 'string') {
        throw new Error('Query must be a non-empty string');
      }
      
      // Set defaults for optional parameters
      const maxResults = params.max_results || 10;
      const documentType = params.document_type || 'all';
      
      // Parse and validate date range if provided
      let dateRange = null;
      if (params.date_range) {
        const { start, end } = params.date_range;
        
        if (start && !isValidDate(start)) {
          throw new Error('Invalid start date format. Use YYYY-MM-DD');
        }
        
        if (end && !isValidDate(end)) {
          throw new Error('Invalid end date format. Use YYYY-MM-DD');
        }
        
        dateRange = {
          start: start ? new Date(start) : null,
          end: end ? new Date(end) : null
        };
      }
      
      // Call document search service
      const results = await searchDocuments({
        query: params.query,
        maxResults,
        documentType,
        dateRange,
        userId: user.id  // Pass user ID for access control
      });
      
      // Return formatted results
      return {
        total_results: results.total,
        results: results.documents.map(doc => ({
          id: doc.id,
          title: doc.title,
          snippet: doc.snippet,
          document_type: doc.type,
          created_date: doc.createdAt,
          updated_date: doc.updatedAt,
          author: doc.author,
          url: doc.url
        }))
      };
    } catch (error) {
      // Log error
      console.error('Document search error:', error);
      
      // Return appropriate error
      throw new Error(`Document search failed: ${error.message}`);
    }
  }
};

// Helper function to validate date format
function isValidDate(dateStr: string): boolean {
  const regex = /^\d{4}-\d{2}-\d{2}$/;
  if (!regex.test(dateStr)) return false;
  
  const date = new Date(dateStr);
  return date instanceof Date && !isNaN(date.getTime());
}

// Register tool
server.registerTool(searchTool);

// Start server
server.start({ port: 8000 })
  .then(() => console.log('Server started on port 8000'))
  .catch(err => console.error('Failed to start server:', err));
```

## Best Practices for Tool Implementation

When implementing tools, follow these best practices:

1. **Design for AI consumption**
   - Clear, detailed descriptions
   - Explicit parameter descriptions
   - Consistent naming conventions
   - Example use cases

2. **Follow security principles**
   - Input validation
   - Output sanitization
   - Proper authentication
   - Secure external calls

3. **Implement robust error handling**
   - Validate inputs early
   - Handle external service failures
   - Provide helpful error messages
   - Include error details for debugging

4. **Optimize performance**
   - Implement caching where appropriate
   - Set reasonable timeouts
   - Monitor resource usage
   - Handle concurrent requests

5. **Provide comprehensive documentation**
   - Clear purpose statement
   - Parameter documentation
   - Return value documentation
   - Error scenarios

## Summary

Implementing tools in MCP involves defining clear functionality, creating detailed documentation, specifying input/output schemas, implementing robust business logic, and handling errors properly. By following best practices and providing comprehensive documentation, you can create tools that are reliable, secure, and easy for AI models to use effectively.
