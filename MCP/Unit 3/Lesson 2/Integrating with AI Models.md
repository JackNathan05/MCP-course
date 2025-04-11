
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 3/Lesson 2/_Lesson 2|_Lesson 2]]

---

## Overview

This lesson focuses on connecting MCP clients with AI models to enable intelligent tool usage.

## Translating MCP Tools to Model Function Calls

AI models typically use function calling or similar mechanisms to interact with external tools. Translating MCP tool definitions to function schemas involves:

1. **Converting MCP tool definitions to function schemas**
   - Mapping tool IDs to function names
   - Converting parameter definitions
   - Preserving descriptions
   - Including validation constraints

2. **Formatting parameters correctly**
   - Ensuring type compatibility
   - Preserving required/optional status
   - Including default values
   - Adding parameter constraints

3. **Handling type conversions**
   - Primitive types (string, number, boolean)
   - Complex types (object, array)
   - Enumerations
   - Custom formats

4. **Providing clear descriptions**
   - Explaining tool purpose
   - Describing parameter usage
   - Documenting return values
   - Explaining error scenarios

5. **Including examples when possible**
   - Sample invocations
   - Expected results
   - Common use cases
   - Error examples

Here's an example of translating an MCP tool to an OpenAI function definition:

```javascript
// Original MCP tool definition
const mcpTool = {
  id: "weather.forecast",
  name: "Weather Forecast",
  description: "Get weather forecast for a specific location",
  parameters: [
    {
      name: "location",
      description: "City name or geographic coordinates",
      type: "string",
      required: true
    },
    {
      name: "days",
      description: "Number of days to forecast (1-10)",
      type: "integer",
      minimum: 1,
      maximum: 10,
      default: 3,
      required: false
    }
  ]
};

// Translated OpenAI function definition
const openAiFunction = {
  name: "weather_forecast",  // Converted to snake_case for OpenAI
  description: "Get weather forecast for a specific location",
  parameters: {
    type: "object",
    properties: {
      location: {
        type: "string",
        description: "City name or geographic coordinates"
      },
      days: {
        type: "integer",
        description: "Number of days to forecast (1-10)",
        minimum: 1,
        maximum: 10,
        default: 3
      }
    },
    required: ["location"]
  }
};
```

Similarly, for Anthropic's Claude:

```javascript
// Translated Claude function definition
const claudeFunction = {
  name: "weather_forecast",
  description: "Get weather forecast for a specific location",
  parameters: {
    type: "object",
    properties: {
      location: {
        type: "string",
        description: "City name or geographic coordinates"
      },
      days: {
        type: "integer",
        description: "Number of days to forecast (1-10)",
        minimum: 1,
        maximum: 10,
        default: 3
      }
    },
    required: ["location"]
  }
};
```

## Passing Context to AI Models Effectively

Context management is critical for AI models to make informed decisions:

1. **Including relevant resource data**
   - Provide only necessary context
   - Format context appropriately
   - Structure information for easy parsing
   - Include metadata for context awareness

2. **Formatting context for model consumption**
   - Use model-specific formats
   - Provide hierarchical structure
   - Label different context types
   - Maintain consistent formatting

3. **Managing context size limits**
   - Prioritize important information
   - Summarize when necessary
   - Truncate least relevant information
   - Use chunking for large contexts

4. **Prioritizing important information**
   - Place critical details first
   - Highlight key decision factors
   - Order information by relevance
   - Include recency indicators

5. **Handling context refresh**
   - Update context when new information arrives
   - Maintain context across interactions
   - Expire outdated context
   - Merge new and existing context

Here's an example of preparing context for an AI model:

```python
def prepare_context_for_model(user_query, mcp_resources, conversation_history, model_context_limit=8000):
    """
    Prepare context for an AI model using MCP resources and conversation history.
    
    Args:
        user_query: Current user query
        mcp_resources: Dictionary of resource results from MCP
        conversation_history: List of previous interactions
        model_context_limit: Maximum context size in tokens (approximate)
    
    Returns:
        Formatted context string for the AI model
    """
    # Start with most important context
    context_parts = [
        f"# User Query\n{user_query}\n\n"
    ]
    
    # Add relevant resources
    if mcp_resources:
        context_parts.append("# Available Information\n")
        
        # Add user profile if available
        if "user_profile" in mcp_resources:
            profile = mcp_resources["user_profile"]
            context_parts.append(f"## User Profile\n")
            context_parts.append(f"Name: {profile.get('name', 'Unknown')}\n")
            context_parts.append(f"Preferences: {', '.join(profile.get('preferences', []))}\n\n")
        
        # Add document content if available
        if "document_content" in mcp_resources:
            docs = mcp_resources["document_content"]
            context_parts.append(f"## Relevant Documents\n")
            for i, doc in enumerate(docs):
                context_parts.append(f"Document {i+1}: {doc.get('title', 'Untitled')}\n")
                context_parts.append(f"{doc.get('content', 'No content available')[:1000]}...\n\n")
        
        # Add system settings if available
        if "system_settings" in mcp_resources:
            settings = mcp_resources["system_settings"]
            context_parts.append(f"## System Settings\n")
            for key, value in settings.items():
                context_parts.append(f"{key}: {value}\n")
            context_parts.append("\n")
    
    # Add conversation history (most recent first)
    if conversation_history:
        context_parts.append("# Conversation History\n")
        # Reverse to get most recent conversations first
        for i, exchange in enumerate(reversed(conversation_history[-5:])):
            context_parts.append(f"User: {exchange['user']}\n")
            context_parts.append(f"Assistant: {exchange['assistant']}\n\n")
    
    # Add available tools
    context_parts.append("# Instructions\n")
    context_parts.append("You can use the following tools to help answer the user's query:\n")
    context_parts.append("1. weather_forecast - Get weather forecast for a location\n")
    context_parts.append("2. calendar_events - Retrieve upcoming calendar events\n")
    context_parts.append("3. search_documents - Search for documents by keyword\n\n")
    
    # Combine all context parts
    full_context = "".join(context_parts)
    
    # Truncate if too long (simplified token counting)
    if len(full_context) > model_context_limit * 4:  # Rough char to token ratio
        # Keep the important parts (query, instructions) and truncate the rest
        important_parts = context_parts[0] + context_parts[-1]  # Query and instructions
        remaining_budget = model_context_limit * 4 - len(important_parts)
        truncated_parts = truncate_content(context_parts[1:-1], remaining_budget)
        full_context = important_parts + truncated_parts
    
    return full_context

def truncate_content(context_parts, max_length):
    """
    Intelligently truncate context parts to fit within max_length.
    Prioritizes more important or recent information.
    """
    result = ""
    current_length = 0
    
    for part in context_parts:
        # If adding this part would exceed the limit
        if current_length + len(part) > max_length:
            # Calculate remaining space
            remaining = max_length - current_length
            if remaining > 100:  # Only add if we have reasonable space
                # Add truncated part with indication
                result += part[:remaining] + "... [truncated]"
            break
        else:
            # Add complete part
            result += part
            current_length += len(part)
    
    return result
```

## Handling Model Responses

Processing AI model outputs is a key part of MCP integration:

1. **Parsing function call requests**
   - Extract function name
   - Parse parameter values
   - Validate against schema
   - Handle multiple function calls

2. **Validating against tool schemas**
   - Check parameter types
   - Verify required parameters
   - Apply validation constraints
   - Handle invalid parameters

3. **Executing requested tools**
   - Map to appropriate MCP tool
   - Format parameters correctly
   - Invoke tool via MCP client
   - Handle execution errors

4. **Formatting responses for the model**
   - Structure tool results
   - Format for model consumption
   - Include error information
   - Provide execution metadata

Here's an example of handling function calls from an OpenAI model:

```python
async def handle_openai_function_calls(model_response, mcp_client):
    """
    Handle function calls from an OpenAI model response.
    
    Args:
        model_response: Response from OpenAI API
        mcp_client: MCP client instance
    
    Returns:
        Processed result to send back to the model
    """
    # Check if the model wants to call a function
    message = model_response.choices[0].message
    
    if not hasattr(message, 'function_call') or message.function_call is None:
        # No function call requested
        return None
    
    try:
        # Extract function call details
        function_name = message.function_call.name
        arguments = json.loads(message.function_call.arguments)
        
        # Map OpenAI function name to MCP tool ID
        tool_id_mapping = {
            "weather_forecast": "weather.forecast",
            "calendar_events": "calendar.events",
            "search_documents": "document.search"
        }
        
        if function_name not in tool_id_mapping:
            raise ValueError(f"Unknown function: {function_name}")
        
        mcp_tool_id = tool_id_mapping[function_name]
        
        # Log the function call
        logging.info(f"Model requested function: {function_name}")
        logging.info(f"Arguments: {arguments}")
        
        # Invoke the MCP tool
        tool_result = await mcp_client.invoke_tool(mcp_tool_id, arguments)
        
        if tool_result is None:
            raise Exception(f"Tool execution failed for {mcp_tool_id}")
        
        # Format the result for the model
        return {
            "role": "function",
            "name": function_name,
            "content": json.dumps(tool_result)
        }
        
    except json.JSONDecodeError:
        logging.error(f"Invalid function arguments: {message.function_call.arguments}")
        return {
            "role": "function",
            "name": message.function_call.name,
            "content": json.dumps({
                "error": "Invalid function arguments: Could not parse JSON"
            })
        }
        
    except Exception as e:
        logging.error(f"Function execution error: {str(e)}")
        return {
            "role": "function",
            "name": message.function_call.name if hasattr(message, 'function_call') else "unknown",
            "content": json.dumps({
                "error": f"Function execution failed: {str(e)}"
            })
        }
```

Similarly for Anthropic's Claude:

```python
async def handle_claude_tool_calls(model_response, mcp_client):
    """
    Handle tool calls from a Claude API response.
    
    Args:
        model_response: Response from Claude API
        mcp_client: MCP client instance
    
    Returns:
        Processed result to send back to the model
    """
    # Extract tool calls from the response
    content = model_response.get('content', [])
    tool_calls = []
    
    for item in content:
        if item.get('type') == 'tool_use':
            tool_calls.append(item.get('tool_use', {}))
    
    if not tool_calls:
        # No tool calls requested
        return None
    
    tool_results = []
    
    for tool_call in tool_calls:
        try:
            # Extract tool call details
            tool_name = tool_call.get('name')
            tool_input = tool_call.get('input', {})
            tool_id = tool_call.get('id')
            
            # Map Claude tool name to MCP tool ID
            tool_id_mapping = {
                "weather_forecast": "weather.forecast",
                "calendar_events": "calendar.events",
                "search_documents": "document.search"
            }
            
            if tool_name not in tool_id_mapping:
                raise ValueError(f"Unknown tool: {tool_name}")
            
            mcp_tool_id = tool_id_mapping[tool_name]
            
            # Log the tool call
            logging.info(f"Model requested tool: {tool_name}")
            logging.info(f"Input: {tool_input}")
            
            # Invoke the MCP tool
            result = await mcp_client.invoke_tool(mcp_tool_id, tool_input)
            
            if result is None:
                raise Exception(f"Tool execution failed for {mcp_tool_id}")
            
            # Format the result for the model
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": tool_id,
                "content": result
            })
            
        except Exception as e:
            logging.error(f"Tool execution error: {str(e)}")
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": tool_id if 'tool_id' in locals() else "unknown",
                "content": {
                    "error": f"Tool execution failed: {str(e)}"
                }
            })
    
    return tool_results
```

## Implementing Tool Selection Logic

Helping models make appropriate tool selections:

1. **Clear tool descriptions**
   - Provide detailed descriptions
   - Explain when to use each tool
   - Specify limitations and constraints
   - Include usage examples

2. **Contextual hints**
   - Provide relevant context for decisions
   - Highlight available information
   - Suggest appropriate tools
   - Include usage patterns

3. **Tool categorization**
   - Group related tools
   - Create hierarchical organization
   - Use consistent naming conventions
   - Apply appropriate tags

4. **Usage examples**
   - Show common use cases
   - Demonstrate parameter usage
   - Include expected results
   - Show error scenarios

5. **Error feedback**
   - Provide clear error information
   - Suggest corrections
   - Explain failure reasons
   - Offer alternatives

Here's an example of how to provide tool selection guidance to a model:

```javascript
// Tool selection guidance for model
const toolSelectionGuidance = `
# Available Tools

You have access to the following tools to help address user queries:

## Information Retrieval

1. search_documents
   - Use when: The user asks about specific information that might be in their documents
   - Example: "Find information about the quarterly budget" or "Look for files about project X"
   - Do NOT use for: General web searches or accessing external information

2. weather_forecast
   - Use when: The user asks about weather conditions for a specific location
   - Example: "What's the weather like in New York?" or "Will it rain tomorrow in London?"
   - Do NOT use for: Historical weather data or long-term climate information

## Personal Information Management

3. calendar_events
   - Use when: The user asks about their schedule or events
   - Example: "What meetings do I have today?" or "When is my next appointment?"
   - Do NOT use for: Creating or modifying events (use calendar_create_event instead)

4. calendar_create_event
   - Use when: The user wants to add an event to their calendar
   - Example: "Schedule a meeting with John tomorrow at 2 PM" or "Add dentist appointment for Friday"
   - Do NOT use for: Retrieving existing events

## Task Management

5. task_list
   - Use when: The user asks about their tasks or to-do items
   - Example: "Show me my tasks" or "What's on my to-do list?"
   - Do NOT use for: Creating new tasks

6. task_create
   - Use when: The user wants to add a new task
   - Example: "Add a task to call mom" or "Create a to-do item for the project proposal"
   - Do NOT use for: Listing or querying existing tasks

# Tool Selection Guidelines

1. Choose the most specific tool for the job
2. Only call tools when necessary to answer the user's query
3. Use only one tool at a time unless absolutely necessary
4. Check the results of one tool before deciding to use another
5. If no tool seems appropriate, respond directly to the user without calling a tool

# Common Error Patterns

1. Using search_documents for general knowledge questions (respond directly instead)
2. Using calendar_events when the user wants to create an event (use calendar_create_event)
3. Using task_list when the user wants to add a task (use task_create)
4. Using weather_forecast for historical weather (explain the limitation)
`;
```

## Complete Integration Example

Let's put it all together with a complete example of integrating MCP with OpenAI:

```python
import asyncio
import json
import logging
from openai import OpenAI
from mcp.client import MCPClient

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class AI_MCP_Integration:
    def __init__(self, mcp_server_url, openai_api_key, model="gpt-4"):
        # Initialize MCP client
        self.mcp_client = MCPClient()
        self.mcp_server_url = mcp_server_url
        self.connected = False
        
        # Initialize OpenAI client
        self.openai_client = OpenAI(api_key=openai_api_key)
        self.model = model
        
        # Initialize state
        self.conversation_history = []
        self.available_tools = {}
        self.function_definitions = []
    
    async def connect(self):
        """Connect to MCP server and discover capabilities"""
        try:
            # Connect to MCP server
            await self.mcp_client.connect(self.mcp_server_url)
            self.connected = True
            logger.info(f"Connected to MCP server at {self.mcp_server_url}")
            
            # Discover capabilities
            capabilities = await self.mcp_client.get_capabilities()
            
            # Extract tools
            self.available_tools = {
                cap.id: cap for cap in capabilities if cap.type == "tool"
            }
            
            # Convert MCP tools to OpenAI function definitions
            self.function_definitions = [
                self._convert_tool_to_function(tool)
                for tool in self.available_tools.values()
            ]
            
            logger.info(f"Discovered {len(self.available_tools)} tools")
            return True
        
        except Exception as e:
            logger.error(f"Failed to connect to MCP server: {str(e)}")
            self.connected = False
            return False
    
    def _convert_tool_to_function(self, tool):
        """Convert MCP tool to OpenAI function definition"""
        # Convert tool ID to valid function name (snake_case)
        function_name = tool.id.replace(".", "_")
        
        # Create properties dictionary for parameters
        properties = {}
        required = []
        
        for param in tool.parameters:
            param_def = {
                "type": param.type,
                "description": param.description
            }
            
            # Add additional properties if available
            if hasattr(param, "enum") and param.enum:
                param_def["enum"] = param.enum
            
            if hasattr(param, "minimum"):
                param_def["minimum"] = param.minimum
                
            if hasattr(param, "maximum"):
                param_def["maximum"] = param.maximum
                
            if hasattr(param, "default") and param.default is not None:
                param_def["default"] = param.default
            
            properties[param.name] = param_def
            
            if param.required:
                required.append(param.name)
        
        # Create function definition
        return {
            "name": function_name,
            "description": tool.description,
            "parameters": {
                "type": "object",
                "properties": properties,
                "required": required
            }
        }
    
    async def process_user_query(self, user_query, context_resources=None):
        """
        Process a user query using AI model and MCP tools.
        
        Args:
            user_query: User's question or request
            context_resources: Dictionary of MCP resources to provide context
        
        Returns:
            AI response with tool results incorporated
        """
        if not self.connected:
            logger.error("Not connected to MCP server")
            return "I'm having trouble connecting to required services. Please try again later."
        
        try:
            # Prepare initial messages
            messages = [
                {"role": "system", "content": "You are a helpful assistant with access to various tools."}
            ]
            
            # Add conversation history
            for exchange in self.conversation_history[-5:]:  # Last 5 exchanges
                messages.append({"role": "user", "content": exchange["user"]})
                messages.append({"role": "assistant", "content": exchange["assistant"]})
            
            # Add current query
            messages.append({"role": "user", "content": user_query})
            
            # Get initial response from model
            response = self.openai_client.chat.completions.create(
                model=self.model,
                messages=messages,
                functions=self.function_definitions,
                function_call="auto"
            )
            
            # Check if model wants to call a function
            message = response.choices[0].message
            
            if hasattr(message, 'function_call') and message.function_call is not None:
                # Extract function call details
                function_name = message.function_call.name
                arguments = json.loads(message.function_call.arguments)
                
                # Map OpenAI function name to MCP tool ID
                mcp_tool_id = function_name.replace("_", ".")
                
                if mcp_tool_id not in self.available_tools:
                    raise ValueError(f"Unknown tool: {mcp_tool_id}")
                
                # Log the function call
                logger.info(f"Model requested tool: {function_name}")
                logger.info(f"Arguments: {arguments}")
                
                # Invoke the MCP tool
                tool_result = await self.mcp_client.invoke_tool(mcp_tool_id, arguments)
                
                # Add function call and result to messages
                messages.append({
                    "role": "assistant",
                    "content": None,
                    "function_call": {
                        "name": function_name,
                        "arguments": message.function_call.arguments
                    }
                })
                
                messages.append({
                    "role": "function",
                    "name": function_name,
                    "content": json.dumps(tool_result)
                })
                
                # Get final response from model
                final_response = self.openai_client.chat.completions.create(
                    model=self.model,
                    messages=messages
                )
                
                assistant_response = final_response.choices[0].message.content
            else:
                # Model didn't need to call a function
                assistant_response = message.content
            
            # Update conversation history
            self.conversation_history.append({
                "user": user_query,
                "assistant": assistant_response
            })
            
            return assistant_response
            
        except Exception as e:
            logger.error(f"Error processing query: {str(e)}")
            return f"I encountered an error while processing your request: {str(e)}"
    
    async def disconnect(self):
        """Disconnect from MCP server"""
        if self.connected:
            try:
                await self.mcp_client.disconnect()
                logger.info("Disconnected from MCP server")
            except Exception as e:
                logger.error(f"Error disconnecting from MCP server: {str(e)}")
            finally:
                self.connected = False

# Example usage
async def main():
    # Initialize integration
    integration = AI_MCP_Integration(
        mcp_server_url="http://localhost:8000",
        openai_api_key="your_openai_api_key",
        model="gpt-4"
    )
    
    # Connect to MCP server
    if await integration.connect():
        try:
            # Process user query
            response = await integration.process_user_query(
                "What's the weather like in San Francisco today?"
            )
            
            print("AI Response:")
            print(response)
            
        finally:
            # Disconnect
            await integration.disconnect()
    else:
        print("Failed to connect to MCP server")

if __name__ == "__main__":
    asyncio.run(main())
```

## Best Practices for AI Model Integration

When integrating MCP with AI models, follow these best practices:

1. **Provide Clear Tool Descriptions**
   - Write detailed, specific descriptions
   - Explain when to use (and not use) each tool
   - Include parameter descriptions
   - Document expected results

2. **Manage Context Effectively**
   - Send only relevant context
   - Prioritize important information
   - Structure context for easy consumption
   - Respect context limits

3. **Handle Errors Gracefully**
   - Validate inputs before execution
   - Provide helpful error messages
   - Include recovery options
   - Return partial results when possible

4. **Maintain Conversation Context**
   - Include relevant conversation history
   - Track tool usage across turns
   - Provide continuity in responses
   - Maintain user preferences

5. **Implement Proper Security**
   - Validate model requests
   - Enforce permission boundaries
   - Implement rate limiting
   - Log all tool invocations

## Summary

Integrating MCP with AI models involves translating tool definitions to function schemas, providing effective context, handling model responses, and implementing tool selection logic. By following best practices for integration, you can create powerful applications that combine the reasoning capabilities of AI models with the functional capabilities of MCP tools.

In the next lesson, we'll explore how to design effective user experiences for applications using MCP and AI models.
