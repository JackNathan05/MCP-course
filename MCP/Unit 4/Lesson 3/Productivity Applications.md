
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 4/Lesson 3/_Lesson 3|_Lesson 3]]

---

## Overview

This lesson explores integrating MCP with productivity applications like chat interfaces, email clients, and task management tools.

## MCP in Chat Interfaces and Assistants

Effective chat integration patterns:

1. **Conversation context management**
   - Conversation history tracking
   - Context persistence
   - Thread management
   - Multi-turn interactions

2. **Tool discovery and suggestion**
   - Contextual tool recommendations
   - Command discovery
   - Auto-complete suggestions
   - Feature education

3. **Natural invocation patterns**
   - Intent recognition
   - Command parsing
   - Parameter extraction
   - Error correction

4. **Response formatting**
   - Structured responses
   - Rich media
   - Interactive elements
   - Consistent styling

5. **Error recovery in conversation**
   - Graceful error handling
   - Suggestion correction
   - Intent clarification
   - Contextual help

MCP enables chat interfaces to provide more powerful and consistent capabilities by standardizing tool integration.

## Example: Chat Interface with MCP Integration

Here's an example of integrating MCP with a chat interface:

```javascript
// Chat interface with MCP integration in React
import React, { useState, useEffect, useRef } from 'react';
import { MCPClient } from 'mcp-client';
import { OpenAI } from 'openai';

// Component for chat interface with MCP integration
const ChatInterface = ({ apiKey }) => {
  // State
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [mcpClient, setMcpClient] = useState(null);
  const [openai, setOpenai] = useState(null);
  const [mcpTools, setMcpTools] = useState([]);
  const [isConnected, setIsConnected] = useState(false);
  
  const messagesEndRef = useRef(null);
  
  // Initialize clients
  useEffect(() => {
    const initClients = async () => {
      // Initialize OpenAI client
      const openaiClient = new OpenAI({ apiKey });
      setOpenai(openaiClient);
      
      // Initialize MCP client
      const mcp = new MCPClient();
      setMcpClient(mcp);
      
      try {
        // Connect to MCP server
        await mcp.connect('http://localhost:8000');
        setIsConnected(true);
        
        // Get available tools
        const capabilities = await mcp.getCapabilities();
        const tools = capabilities.filter(cap => cap.type === 'tool');
        setMcpTools(tools);
        
        // Add welcome message
        setMessages([
          {
            role: 'system',
            content: 'Welcome! I can help you with various tasks. What would you like to do?'
          }
        ]);
      } catch (error) {
        console.error('Failed to connect to MCP server:', error);
        setIsConnected(false);
        
        // Add error message
        setMessages([
          {
            role: 'system',
            content: 'I encountered an issue connecting to some services. Some features might be limited.',
            isError: true
          }
        ]);
      }
    };
    
    initClients();
    
    // Cleanup on unmount
    return () => {
      if (mcpClient && isConnected) {
        mcpClient.disconnect().catch(console.error);
      }
    };
  }, [apiKey]);
  
  // Scroll to bottom when messages change
  useEffect(() => {
    if (messagesEndRef.current) {
      messagesEndRef.current.scrollIntoView({ behavior: 'smooth' });
    }
  }, [messages]);
  
  // Convert MCP tools to OpenAI functions
  const convertToolsToFunctions = (tools) => {
    return tools.map(tool => {
      const properties = {};
      const required = [];
      
      // Convert parameters
      tool.parameters.forEach(param => {
        properties[param.name] = {
          type: param.type,
          description: param.description
        };
        
        // Add enum if available
        if (param.enum) {
          properties[param.name].enum = param.enum;
        }
        
        // Add min/max if available
        if (param.minimum !== undefined) {
          properties[param.name].minimum = param.minimum;
        }
        if (param.maximum !== undefined) {
          properties[param.name].maximum = param.maximum;
        }
        
        // Add default if available
        if (param.default !== undefined) {
          properties[param.name].default = param.default;
        }
        
        // Track required parameters
        if (param.required) {
          required.push(param.name);
        }
      });
      
      // Return function definition
      return {
        name: tool.id.replace(/\./g, '_'),  // Replace dots with underscores
        description: tool.description,
        parameters: {
          type: 'object',
          properties,
          required
        }
      };
    });
  };
  
  // Handle sending a message
  const handleSendMessage = async () => {
    if (!input.trim() || isLoading) return;
    
    // Add user message
    const userMessage = { role: 'user', content: input };
    setMessages(prev => [...prev, userMessage]);
    setInput('');
    setIsLoading(true);
    
    try {
      // Check if connected to MCP
      if (!isConnected || !mcpClient || !openai) {
        throw new Error('Not connected to required services');
      }
      
      // Get history for context (last 5 messages)
      const history = messages.slice(-5).map(msg => ({
        role: msg.role,
        content: msg.content
      }));
      
      // Convert MCP tools to OpenAI functions
      const functions = convertToolsToFunctions(mcpTools);
      
      // Add thinking message
      setMessages(prev => [...prev, { role: 'assistant', content: '...', isThinking: true }]);
      
      // Call OpenAI API
      const completion = await openai.chat.completions.create({
        model: 'gpt-4',
        messages: [...history, userMessage],
        functions,
        function_call: 'auto'
      });
      
      // Remove thinking message
      setMessages(prev => prev.filter(msg => !msg.isThinking));
      
      const response = completion.choices[0].message;
      
      // Check if model wants to call a function
      if (response.function_call) {
        // Get function details
        const { name, arguments: args } = response.function_call;
        
        // Parse arguments
        const parsedArgs = JSON.parse(args);
        
        // Convert function name back to MCP tool ID
        const toolId = name.replace(/_/g, '.');
        
        // Find tool
        const tool = mcpTools.find(t => t.id === toolId);
        
        if (!tool) {
          throw new Error(`Unknown tool: ${toolId}`);
        }
        
        // Add function call message
        setMessages(prev => [
          ...prev,
          {
            role: 'assistant',
            content: `I'll help you with that using ${tool.name}...`,
            toolIntent: true
          }
        ]);
        
        // Add tool execution message
        setMessages(prev => [
          ...prev,
          {
            role: 'tool',
            toolId,
            toolName: tool.name,
            parameters: parsedArgs
          }
        ]);
        
        try {
          // Call MCP tool
          const result = await mcpClient.invokeTool(toolId, parsedArgs);
          
          // Add tool result message
          setMessages(prev => [
            ...prev,
            {
              role: 'tool-result',
              toolId,
              result
            }
          ]);
          
          // Get final response with tool result
          const finalCompletion = await openai.chat.completions.create({
            model: 'gpt-4',
            messages: [
              ...history,
              userMessage,
              {
                role: 'assistant',
                content: null,
                function_call: {
                  name,
                  arguments: args
                }
              },
              {
                role: 'function',
                name,
                content: JSON.stringify(result)
              }
            ]
          });
          
          // Add final assistant message
          setMessages(prev => [
            ...prev,
            {
              role: 'assistant',
              content: finalCompletion.choices[0].message.content
            }
          ]);
          
        } catch (toolError) {
          console.error('Tool execution error:', toolError);
          
          // Add tool error message
          setMessages(prev => [
            ...prev,
            {
              role: 'tool-error',
              toolId,
              error: toolError.message
            }
          ]);
          
          // Get error response
          const errorCompletion = await openai.chat.completions.create({
            model: 'gpt-4',
            messages: [
              ...history,
              userMessage,
              {
                role: 'assistant',
                content: null,
                function_call: {
                  name,
                  arguments: args
                }
              },
              {
                role: 'function',
                name,
                content: JSON.stringify({ error: toolError.message })
              }
            ]
          });
          
          // Add error response message
          setMessages(prev => [
            ...prev,
            {
              role: 'assistant',
              content: errorCompletion.choices[0].message.content,
              isError: true
            }
          ]);
        }
      } else {
        // Add regular assistant response
        setMessages(prev => [
          ...prev,
          {
            role: 'assistant',
            content: response.content
          }
        ]);
      }
    } catch (error) {
      console.error('Error processing message:', error);
      
      // Remove thinking message if still present
      setMessages(prev => prev.filter(msg => !msg.isThinking));
      
      // Add error message
      setMessages(prev => [
        ...prev,
        {
          role: 'assistant',
          content: `I encountered an error: ${error.message}`,
          isError: true
        }
      ]);
    } finally {
      setIsLoading(false);
    }
  };
  
  // Render message
  const renderMessage = (message, index) => {
    switch (message.role) {
      case 'user':
        return (
          <div key={index} className="message user-message">
            <div className="message-avatar">You</div>
            <div className="message-content">{message.content}</div>
          </div>
        );
        
      case 'assistant':
        return (
          <div key={index} className={`message assistant-message ${message.isError ? 'error' : ''}`}>
            <div className="message-avatar">AI</div>
            <div className="message-content">
              {message.isThinking ? (
                <div className="thinking-indicator">
                  <span className="dot"></span>
                  <span className="dot"></span>
                  <span className="dot"></span>
                </div>
              ) : (
                message.content
              )}
            </div>
          </div>
        );
        
      case 'system':
        return (
          <div key={index} className={`message system-message ${message.isError ? 'error' : ''}`}>
            <div className="message-content">{message.content}</div>
          </div>
        );
        
      case 'tool':
        return (
          <div key={index} className="message tool-message">
            <div className="tool-header">
              <span className="tool-icon">🔧</span>
              <span className="tool-name">{message.toolName}</span>
            </div>
            <div className="tool-parameters">
              {Object.entries(message.parameters).map(([key, value]) => (
                <div key={key} className="parameter">
                  <span className="parameter-name">{key}:</span>
                  <span className="parameter-value">
                    {typeof value === 'object' ? JSON.stringify(value) : String(value)}
                  </span>
                </div>
              ))}
            </div>
          </div>
        );
        
      case 'tool-result':
        return (
          <div key={index} className="message tool-result-message">
            <div className="tool-result">
              {typeof message.result === 'object' ? (
                <pre>{JSON.stringify(message.result, null, 2)}</pre>
              ) : (
                String(message.result)
              )}
            </div>
          </div>
        );
        
      case 'tool-error':
        return (
          <div key={index} className="message tool-error-message">
            <div className="tool-error">
              <span className="error-icon">⚠️</span>
              <span className="error-message">{message.error}</span>
            </div>
          </div>
        );
        
      default:
        return null;
    }
  };
  
  return (
    <div className="chat-interface">
      <div className="chat-header">
        <h2>AI Assistant</h2>
        <div className={`connection-status ${isConnected ? 'connected' : 'disconnected'}`}>
          {isConnected ? 'Connected' : 'Disconnected'}
        </div>
      </div>
      
      <div className="chat-messages">
        {messages.map(renderMessage)}
        <div ref={messagesEndRef} />
      </div>
      
      <div className="chat-input">
        <textarea
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type a message..."
          disabled={isLoading}
          onKeyDown={(e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
              e.preventDefault();
              handleSendMessage();
            }
          }}
        />
        <button
          onClick={handleSendMessage}
          disabled={isLoading || !input.trim()}
        >
          {isLoading ? 'Sending...' : 'Send'}
        </button>
      </div>
    </div>
  );
};

export default ChatInterface;
```

## Calendar and Email Integration Patterns

Common productivity integrations:

1. **Email composition and analysis**
   - Email drafting
   - Reply suggestions
   - Summary generation
   - Priority identification

2. **Meeting scheduling and management**
   - Availability checking
   - Meeting creation
   - Scheduling optimization
   - Attendee coordination

3. **Calendar availability checking**
   - Free/busy lookup
   - Time slot suggestion
   - Scheduling constraints
   - Time zone handling

4. **Email classification and routing**
   - Importance classification
   - Folder organization
   - Automated replies
   - Follow-up reminders

5. **Notification management**
   - Priority filtering
   - Digests creation
   - Delivery timing
   - Channel selection

MCP enables standardized access to email and calendar systems, allowing AI assistants to help with common productivity tasks.

## Example: Calendar Integration with MCP

Here's an example of a calendar MCP server:

```python
# Calendar MCP server
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter, MCPResource
import os
import datetime
import json
import logging
from typing import List, Dict, Any, Optional

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Mock calendar database
class CalendarDB:
    def __init__(self):
        self.events = []
        self.load_sample_events()
    
    def load_sample_events(self):
        # Sample events for demonstration
        self.events = [
            {
                "id": "event-001",
                "title": "Team Meeting",
                "start": "2025-04-10T10:00:00Z",
                "end": "2025-04-10T11:00:00Z",
                "location": "Conference Room A",
                "attendees": ["user1@example.com", "user2@example.com", "user3@example.com"],
                "description": "Weekly team sync"
            },
            {
                "id": "event-002",
                "title": "Project Review",
                "start": "2025-04-10T14:00:00Z",
                "end": "2025-04-10T15:30:00Z",
                "location": "Virtual Meeting",
                "attendees": ["user1@example.com", "user4@example.com"],
                "description": "Review project progress and next steps"
            },
            {
                "id": "event-003",
                "title": "Client Call",
                "start": "2025-04-11T09:00:00Z",
                "end": "2025-04-11T10:00:00Z",
                "location": "Phone",
                "attendees": ["user1@example.com", "client@company.com"],
                "description": "Discuss contract renewal"
            }
        ]
    
    def get_events(self, start_date=None, end_date=None, user=None):
        """Get calendar events with optional filtering"""
        filtered_events = self.events.copy()
        
        # Filter by date range
        if start_date:
            filtered_events = [e for e in filtered_events if e["start"] >= start_date]
        
        if end_date:
            filtered_events = [e for e in filtered_events if e["start"] <= end_date]
        
        # Filter by user
        if user:
            filtered_events = [e for e in filtered_events if user in e["attendees"]]
        
        return filtered_events
    
    def get_event(self, event_id):
        """Get a specific event by ID"""
        for event in self.events:
            if event["id"] == event_id:
                return event
        return None
    
    def create_event(self, event_data):
        """Create a new calendar event"""
        # Generate event ID
        event_id = f"event-{len(self.events) + 1:03d}"
        
        # Create event
        event = {
            "id": event_id,
            **event_data
        }
        
        # Add to events
        self.events.append(event)
        
        return event
    
    def update_event(self, event_id, event_data):
        """Update an existing event"""
        event = self.get_event(event_id)
        
        if not event:
            return None
        
        # Update event fields
        for key, value in event_data.items():
            if key != "id":  # Don't update ID
                event[key] = value
        
        return event
    
    def delete_event(self, event_id):
        """Delete an event"""
        event = self.get_event(event_id)
        
        if not event:
            return False
        
        # Remove event
        self.events = [e for e in self.events if e["id"] != event_id]
        
        return True
    
    def check_availability(self, users, start_time, end_time):
        """Check if users are available during the specified time"""
        # Get events for all users during the time period
        all_events = []
        
        for user in users:
            user_events = [
                e for e in self.events
                if user in e["attendees"]
                and not (e["end"] <= start_time or e["start"] >= end_time)
            ]
            all_events.extend(user_events)
        
        # If no events found, users are available
        if not all_events:
            return {"available": True, "events": []}
        
        # Users are not available
        return {
            "available": False,
            "events": all_events
        }
    
    def find_available_times(self, users, date, duration_minutes, working_hours=None):
        """Find available time slots for a meeting"""
        if working_hours is None:
            working_hours = {"start": "09:00", "end": "17:00"}
        
        # Get start and end of working day
        day_start = f"{date}T{working_hours['start']}:00Z"
        day_end = f"{date}T{working_hours['end']}:00Z"
        
        # Get events for all users during the day
        all_events = []
        for user in users:
            user_events = [
                e for e in self.events
                if user in e["attendees"]
                and e["start"] >= day_start
                and e["start"] <= day_end
            ]
            all_events.extend(user_events)
        
        # Sort events by start time
        all_events.sort(key=lambda e: e["start"])
        
        # Find available slots
        available_slots = []
        current_time = day_start
        
        for event in all_events:
            # Check if there's enough time before this event
            time_before_event = self._time_difference_minutes(current_time, event["start"])
            
            if time_before_event >= duration_minutes:
                available_slots.append({
                    "start": current_time,
                    "end": self._add_minutes(current_time, duration_minutes)
                })
            
            # Update current time to after this event
            current_time = event["end"]
        
        # Check for available time after the last event
        time_after_last = self._time_difference_minutes(current_time, day_end)
        
        if time_after_last >= duration_minutes:
            available_slots.append({
                "start": current_time,
                "end": self._add_minutes(current_time, duration_minutes)
            })
        
        return available_slots
    
    def _time_difference_minutes(self, time1, time2):
        """Calculate time difference in minutes"""
        t1 = datetime.datetime.fromisoformat(time1.replace('Z', '+00:00'))
        t2 = datetime.datetime.fromisoformat(time2.replace('Z', '+00:00'))
        
        diff = t2 - t1
        return diff.total_seconds() / 60
    
    def _add_minutes(self, time_str, minutes):
        """Add minutes to a time string"""
        t = datetime.datetime.fromisoformat(time_str.replace('Z', '+00:00'))
        t = t + datetime.timedelta(minutes=minutes)
        return t.isoformat().replace('+00:00', 'Z')

# Initialize calendar database
calendar_db = CalendarDB()

# Create MCP server
server = MCPServer()

# Register calendar events tool
@server.tool(
    id="calendar.events",
    description="Get calendar events for a specific date range",
    parameters=[
        MCPParameter(
            name="start_date",
            description="Start date (ISO format)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="end_date",
            description="End date (ISO format)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="user",
            description="User email (optional)",
            type="string",
            required=False
        )
    ]
)
async def get_calendar_events(start_date, end_date, user=None):
    """Get calendar events for a specific date range"""
    try:
        events = calendar_db.get_events(start_date, end_date, user)
        
        return {
            "start_date": start_date,
            "end_date": end_date,
            "user": user,
            "events": events
        }
    except Exception as e:
        logger.error(f"Error getting calendar events: {str(e)}")
        raise Exception(f"Failed to get calendar events: {str(e)}")

# Register event creation tool
@server.tool(
    id="calendar.create_event",
    description="Create a new calendar event",
    parameters=[
        MCPParameter(
            name="title",
            description="Event title",
            type="string",
            required=True
        ),
        MCPParameter(
            name="start",
            description="Start time (ISO format)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="end",
            description="End time (ISO format)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="location",
            description="Event location",
            type="string",
            required=False
        ),
        MCPParameter(
            name="attendees",
            description="List of attendee emails",
            type="array",
            items={"type": "string"},
            required=False
        ),
        MCPParameter(
            name="description",
            description="Event description",
            type="string",
            required=False
        )
    ]
)
async def create_calendar_event(title, start, end, location=None, attendees=None, description=None):
    """Create a new calendar event"""
    try:
        event_data = {
            "title": title,
            "start": start,
            "end": end,
            "location": location,
            "attendees": attendees or [],
            "description": description
        }
        
        event = calendar_db.create_event(event_data)
        
        return event
    except Exception as e:
        logger.error(f"Error creating calendar event: {str(e)}")
        raise Exception(f"Failed to create calendar event: {str(e)}")

# Register availability checking tool
@server.tool(
    id="calendar.check_availability",
    description="Check availability for users during a specific time",
    parameters=[
        MCPParameter(
            name="users",
            description="List of user emails",
            type="array",
            items={"type": "string"},
            required=True
        ),
        MCPParameter(
            name="start_time",
            description="Start time (ISO format)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="end_time",
            description="End time (ISO format)",
            type="string",
            required=True
        )
    ]
)
async def check_availability(users, start_time, end_time):
    """Check availability for users during a specific time"""
    try:
        availability = calendar_db.check_availability(users, start_time, end_time)
        
        return {
            "users": users,
            "start_time": start_time,
            "end_time": end_time,
            "available": availability["available"],
            "conflicts": [
                {
                    "id": event["id"],
                    "title": event["title"],
                    "start": event["start"],
                    "end": event["end"]
                }
                for event in availability.get("events", [])
            ]
        }
    except Exception as e:
        logger.error(f"Error checking availability: {str(e)}")
        raise Exception(f"Failed to check availability: {str(e)}")

# Register find available times tool
@server.tool(
    id="calendar.find_available_times",
    description="Find available time slots for a meeting",
    parameters=[
        MCPParameter(
            name="users",
            description="List of user emails",
            type="array",
            items={"type": "string"},
            required=True
        ),
        MCPParameter(
            name="date",
            description="Date (YYYY-MM-DD)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="duration_minutes",
            description="Meeting duration in minutes",
            type="integer",
            minimum=15,
            maximum=480,
            required=True
        ),
        MCPParameter(
            name="working_hours",
            description="Working hours",
            type="object",
            properties={
                "start": {"type": "string"},
                "end": {"type": "string"}
            },
            required=False
        )
    ]
)
async def find_available_times(users, date, duration_minutes, working_hours=None):
    """Find available time slots for a meeting"""
    try:
        available_slots = calendar_db.find_available_times(
            users, date, duration_minutes, working_hours
        )
        
        return {
            "users": users,
            "date": date,
            "duration_minutes": duration_minutes,
            "working_hours": working_hours or {"start": "09:00", "end": "17:00"},
            "available_slots": available_slots
        }
    except Exception as e:
        logger.error(f"Error finding available times: {str(e)}")
        raise Exception(f"Failed to find available times: {str(e)}")

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"Calendar MCP server running on port {port}")
```

## Task Management Workflows

Task-focused integration patterns:

1. **Task creation and tracking**
   - Task definition
   - Status tracking
   - Progress updates
   - Dependency management

2. **Project management**
   - Project organization
   - Timeline management
   - Resource allocation
   - Status reporting

3. **Priority and deadline management**
   - Priority assignment
   - Deadline tracking
   - Scheduling assistance
   - Reminders and notifications

4. **Delegation and assignment**
   - Task assignment
   - Responsibility tracking
   - Handoff workflows
   - Team coordination

5. **Progress reporting**
   - Status updates
   - Milestone tracking
   - Blockers identification
   - Achievement recognition

MCP enables AI assistants to help manage tasks and projects more effectively through standardized integrations.

## Example: Task Management MCP Server

Here's an example of a task management MCP server:

```python
# Task management MCP server
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter, MCPResource
import os
import datetime
import json
import logging
import uuid
from typing import List, Dict, Any, Optional

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Mock task database
class TaskDB:
    def __init__(self):
        self.tasks = []
        self.load_sample_tasks()
    
    def load_sample_tasks(self):
        # Sample tasks for demonstration
        self.tasks = [
            {
                "id": "task-001",
                "title": "Prepare quarterly report",
                "description": "Compile Q1 2025 financial reports for the board meeting",
                "status": "in_progress",
                "priority": "high",
                "due_date": "2025-04-15",
                "assignee": "alice@example.com",
                "created_at": "2025-04-01T09:00:00Z",
                "updated_at": "2025-04-08T14:30:00Z",
                "tags": ["finance", "reporting", "quarterly"]
            },
            {
                "id": "task-002",
                "title": "Review product roadmap",
                "description": "Evaluate and provide feedback on the Q2 product roadmap",
                "status": "not_started",
                "priority": "medium",
                "due_date": "2025-04-20",
                "assignee": "bob@example.com",
                "created_at": "2025-04-05T10:15:00Z",
                "updated_at": "2025-04-05T10:15:00Z",
                "tags": ["product", "planning", "review"]
            },
            {
                "id": "task-003",
                "title": "Fix login page bug",
                "description": "Address the authentication issue reported by users",
                "status": "completed",
                "priority": "high",
                "due_date": "2025-04-07",
                "assignee": "charlie@example.com",
                "created_at": "2025-04-02T11:30:00Z",
                "updated_at": "2025-04-07T16:45:00Z",
                "tags": ["development", "bug", "security"]
            }
        ]
    
    def get_tasks(self, status=None, assignee=None, due_date=None, priority=None, tags=None):
        """Get tasks with optional filtering"""
        filtered_tasks = self.tasks.copy()
        
        # Filter by status
        if status:
            if isinstance(status, list):
                filtered_tasks = [t for t in filtered_tasks if t["status"] in status]
            else:
                filtered_tasks = [t for t in filtered_tasks if t["status"] == status]
        
        # Filter by assignee
        if assignee:
            filtered_tasks = [t for t in filtered_tasks if t["assignee"] == assignee]
        
        # Filter by due date
        if due_date:
            filtered_tasks = [t for t in filtered_tasks if t["due_date"] == due_date]
        
        # Filter by priority
        if priority:
            if isinstance(priority, list):
                filtered_tasks = [t for t in filtered_tasks if t["priority"] in priority]
            else:
                filtered_tasks = [t for t in filtered_tasks if t["priority"] == priority]
        
        # Filter by tags
        if tags:
            filtered_tasks = [
                t for t in filtered_tasks
                if any(tag in t["tags"] for tag in tags)
            ]
        
        return filtered_tasks
    
    def get_task(self, task_id):
        """Get a specific task by ID"""
        for task in self.tasks:
            if task["id"] == task_id:
                return task
        return None
    
    def create_task(self, task_data):
        """Create a new task"""
        # Generate task ID
        task_id = f"task-{len(self.tasks) + 1:03d}"
        
        # Create timestamp
        now = datetime.datetime.utcnow().isoformat() + "Z"
        
        # Create task
        task = {
            "id": task_id,
            "created_at": now,
            "updated_at": now,
            **task_data
        }
        
        # Add to tasks
        self.tasks.append(task)
        
        return task
    
    def update_task(self, task_id, task_data):
        """Update an existing task"""
        task = self.get_task(task_id)
        
        if not task:
            return None
        
        # Update timestamp
        now = datetime.datetime.utcnow().isoformat() + "Z"
        task["updated_at"] = now
        
        # Update task fields
        for key, value in task_data.items():
            if key not in ["id", "created_at", "updated_at"]:
                task[key] = value
        
        return task
    
    def delete_task(self, task_id):
        """Delete a task"""
        task = self.get_task(task_id)
        
        if not task:
            return False
        
        # Remove task
        self.tasks = [t for t in self.tasks if t["id"] != task_id]
        
        return True

# Initialize task database
task_db = TaskDB()

# Create MCP server
server = MCPServer()

# Register get tasks tool
@server.tool(
    id="tasks.list",
    description="Get tasks with optional filtering",
    parameters=[
        MCPParameter(
            name="status",
            description="Filter by status (not_started, in_progress, completed)",
            type="string",
            enum=["not_started", "in_progress", "completed"],
            required=False
        ),
        MCPParameter(
            name="assignee",
            description="Filter by assignee email",
            type="string",
            required=False
        ),
        MCPParameter(
            name="due_date",
            description="Filter by due date (YYYY-MM-DD)",
            type="string",
            required=False
        ),
        MCPParameter(
            name="priority",
            description="Filter by priority (low, medium, high)",
            type="string",
            enum=["low", "medium", "high"],
            required=False
        ),
        MCPParameter(
            name="tags",
            description="Filter by tags",
            type="array",
            items={"type": "string"},
            required=False
        )
    ]
)
async def list_tasks(status=None, assignee=None, due_date=None, priority=None, tags=None):
    """Get tasks with optional filtering"""
    try:
        tasks = task_db.get_tasks(status, assignee, due_date, priority, tags)
        
        # Create filters description
        filters = {}
        if status:
            filters["status"] = status
        if assignee:
            filters["assignee"] = assignee
        if due_date:
            filters["due_date"] = due_date
        if priority:
            filters["priority"] = priority
        if tags:
            filters["tags"] = tags
        
        return {
            "filters": filters,
            "count": len(tasks),
            "tasks": tasks
        }
    except Exception as e:
        logger.error(f"Error listing tasks: {str(e)}")
        raise Exception(f"Failed to list tasks: {str(e)}")

# Register create task tool
@server.tool(
    id="tasks.create",
    description="Create a new task",
    parameters=[
        MCPParameter(
            name="title",
            description="Task title",
            type="string",
            required=True
        ),
        MCPParameter(
            name="description",
            description="Task description",
            type="string",
            required=False
        ),
        MCPParameter(
            name="status",
            description="Task status",
            type="string",
            enum=["not_started", "in_progress", "completed"],
            default="not_started",
            required=False
        ),
        MCPParameter(
            name="priority",
            description="Task priority",
            type="string",
            enum=["low", "medium", "high"],
            default="medium",
            required=False
        ),
        MCPParameter(
            name="due_date",
            description="Due date (YYYY-MM-DD)",
            type="string",
            required=False
        ),
        MCPParameter(
            name="assignee",
            description="Assignee email",
            type="string",
            required=False
        ),
        MCPParameter(
            name="tags",
            description="Task tags",
            type="array",
            items={"type": "string"},
            required=False
        )
    ]
)
async def create_task(title, description=None, status="not_started", priority="medium", due_date=None, assignee=None, tags=None):
    """Create a new task"""
    try:
        task_data = {
            "title": title,
            "description": description,
            "status": status,
            "priority": priority,
            "due_date": due_date,
            "assignee": assignee,
            "tags": tags or []
        }
        
        task = task_db.create_task(task_data)
        
        return task
    except Exception as e:
        logger.error(f"Error creating task: {str(e)}")
        raise Exception(f"Failed to create task: {str(e)}")

# Register update task tool
@server.tool(
    id="tasks.update",
    description="Update an existing task",
    parameters=[
        MCPParameter(
            name="task_id",
            description="Task ID",
            type="string",
            required=True
        ),
        MCPParameter(
            name="title",
            description="Task title",
            type="string",
            required=False
        ),
        MCPParameter(
            name="description",
            description="Task description",
            type="string",
            required=False
        ),
        MCPParameter(
            name="status",
            description="Task status",
            type="string",
            enum=["not_started", "in_progress", "completed"],
            required=False
        ),
        MCPParameter(
            name="priority",
            description="Task priority",
            type="string",
            enum=["low", "medium", "high"],
            required=False
        ),
        MCPParameter(
            name="due_date",
            description="Due date (YYYY-MM-DD)",
            type="string",
            required=False
        ),
        MCPParameter(
            name="assignee",
            description="Assignee email",
            type="string",
            required=False
        ),
        MCPParameter(
            name="tags",
            description="Task tags",
            type="array",
            items={"type": "string"},
            required=False
        )
    ]
)
async def update_task(task_id, **kwargs):
    """Update an existing task"""
    try:
        # Remove None values
        update_data = {k: v for k, v in kwargs.items() if v is not None}
        
        # Update task
        task = task_db.update_task(task_id, update_data)
        
        if not task:
            raise Exception(f"Task not found: {task_id}")
        
        return task
    except Exception as e:
        logger.error(f"Error updating task: {str(e)}")
        raise Exception(f"Failed to update task: {str(e)}")

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"Task management MCP server running on port {port}")
```

## Multi-Tool Orchestration

Combining tools effectively:

1. **Workflow sequencing**
   - Task ordering
   - Dependency management
   - Process automation
   - Sequential execution

2. **Data passing between tools**
   - Result transformation
   - Parameter mapping
   - Context sharing
   - State management

3. **Conditional execution**
   - Decision branching
   - Rule evaluation
   - Dynamic tool selection
   - Error handling logic

4. **Error handling across tools**
   - Error propagation
   - Graceful degradation
   - Recovery strategies
   - Alternative paths

5. **User confirmation points**
   - Interactive checkpoints
   - Progress validation
   - Decision confirmation
   - Result verification

MCP enables complex workflows that involve multiple tools working together seamlessly.

## Example: Multi-Tool Workflow

Here's an example of a multi-tool workflow that helps schedule a meeting:

```javascript
// Multi-tool workflow for scheduling a meeting
async function scheduleMeetingWorkflow(mcpClient, openai, request) {
  const results = {
    workflow: "schedule_meeting",
    status: "in_progress",
    steps: [],
    finalResult: null
  };
  
  try {
    // Step 1: Extract meeting details from request
    results.steps.push({
      step: "extract_meeting_details",
      status: "in_progress"
    });
    
    const meetingDetails = await extractMeetingDetails(openai, request);
    
    results.steps[0].status = "completed";
    results.steps[0].result = meetingDetails;
    
    // Step 2: Check availability of attendees
    results.steps.push({
      step: "check_availability",
      status: "in_progress"
    });
    
    const availability = await checkAttendeeAvailability(mcpClient, meetingDetails);
    
    results.steps[1].status = "completed";
    results.steps[1].result = availability;
    
    // Step 3: Find available time slots
    results.steps.push({
      step: "find_available_slots",
      status: "in_progress"
    });
    
    const availableSlots = await findAvailableTimeSlots(mcpClient, meetingDetails, availability);
    
    results.steps[2].status = "completed";
    results.steps[2].result = availableSlots;
    
    // Step 4: Create calendar event
    results.steps.push({
      step: "create_calendar_event",
      status: "in_progress"
    });
    
    const calendarEvent = await createCalendarEvent(mcpClient, meetingDetails, availableSlots[0]);
    
    results.steps[3].status = "completed";
    results.steps[3].result = calendarEvent;
    
    // Step 5: Create follow-up task
    results.steps.push({
      step: "create_followup_task",
      status: "in_progress"
    });
    
    const followupTask = await createFollowupTask(mcpClient, meetingDetails, calendarEvent);
    
    results.steps[4].status = "completed";
    results.steps[4].result = followupTask;
    
    // Final result
    results.status = "completed";
    results.finalResult = {
      message: "Meeting scheduled successfully",
      event: calendarEvent,
      task: followupTask
    };
    
    return results;
    
  } catch (error) {
    // Handle workflow error
    const failedStepIndex = results.steps.findIndex(step => step.status === "in_progress");
    
    if (failedStepIndex >= 0) {
      results.steps[failedStepIndex].status = "failed";
      results.steps[failedStepIndex].error = error.message;
    }
    
    results.status = "failed";
    results.error = error.message;
    
    return results;
  }
}

// Step 1: Extract meeting details from request
async function extractMeetingDetails(openai, request) {
  const completion = await openai.chat.completions.create({
    model: "gpt-4",
    messages: [
      {
        role: "system",
        content: "Extract meeting details from the user request. Return a JSON object with title, description, attendees, date, duration_minutes, and importance."
      },
      {
        role: "user",
        content: request
      }
    ],
    response_format: { type: "json_object" }
  });
  
  return JSON.parse(completion.choices[0].message.content);
}

// Step 2: Check availability of attendees
async function checkAttendeeAvailability(mcpClient, meetingDetails) {
  const { attendees, date } = meetingDetails;
  
  // Format date
  const formattedDate = date.split('T')[0];
  
  // Get events for all attendees
  const events = await mcpClient.invokeTool("calendar.events", {
    start_date: `${formattedDate}T00:00:00Z`,
    end_date: `${formattedDate}T23:59:59Z`,
    attendees
  });
  
  return {
    date: formattedDate,
    attendees,
    events
  };
}

// Step 3: Find available time slots
async function findAvailableTimeSlots(mcpClient, meetingDetails, availability) {
  const { attendees, date, duration_minutes } = meetingDetails;
  
  // Format date
  const formattedDate = date.split('T')[0];
  
  // Find available slots
  const availableSlots = await mcpClient.invokeTool("calendar.find_available_times", {
    users: attendees,
    date: formattedDate,
    duration_minutes: duration_minutes || 30,
    working_hours: {
      start: "09:00",
      end: "17:00"
    }
  });
  
  if (!availableSlots.available_slots || availableSlots.available_slots.length === 0) {
    throw new Error("No available time slots found for all attendees");
  }
  
  return availableSlots.available_slots;
}

// Step 4: Create calendar event
async function createCalendarEvent(mcpClient, meetingDetails, timeSlot) {
  const { title, description, attendees } = meetingDetails;
  
  // Create event
  const event = await mcpClient.invokeTool("calendar.create_event", {
    title,
    start: timeSlot.start,
    end: timeSlot.end,
    attendees,
    description
  });
  
  return event;
}

// Step 5: Create follow-up task
async function createFollowupTask(mcpClient, meetingDetails, calendarEvent) {
  const { title, importance } = meetingDetails;
  
  // Map importance to priority
  const priorityMap = {
    high: "high",
    medium: "medium",
    low: "low"
  };
  
  // Create follow-up task
  const task = await mcpClient.invokeTool("tasks.create", {
    title: `Prepare for meeting: ${title}`,
    description: `Prepare materials for the meeting scheduled at ${calendarEvent.start}`,
    priority: priorityMap[importance] || "medium",
    due_date: calendarEvent.start.split('T')[0],
    tags: ["meeting", "preparation"]
  });
  
  return task;
}
```

## Best Practices for Productivity Integration

When integrating MCP with productivity applications, follow these best practices:

1. **Contextual Awareness**
   - Maintain user context
   - Consider workflow state
   - Remember previous interactions
   - Adapt to user preferences

2. **Seamless Integration**
   - Consistent user experience
   - Unified interaction model
   - Transparent tool usage
   - Natural transitions

3. **Progressive Enhancement**
   - Start with basic capabilities
   - Add advanced features progressively
   - Adapt to user expertise
   - Provide discovery mechanisms

4. **Privacy Preservation**
   - Minimize data exposure
   - Respect data boundaries
   - Implement proper authentication
   - Handle sensitive information carefully

5. **Feedback Loop**
   - Provide clear status updates
   - Communicate tool actions
   - Offer error explanations
   - Enable user corrections

6. **Flexibility and Customization**
   - Support user preferences
   - Allow workflow adaptation
   - Enable tool customization
   - Respect individual work styles

7. **Performance and Reliability**
   - Optimize response times
   - Handle network issues
   - Implement fault tolerance
   - Ensure consistent behavior

## Summary

MCP enables powerful integrations with productivity applications like chat interfaces, email clients, and task management tools. By standardizing these integrations, AI assistants can help users with a wide range of productivity tasks, from scheduling meetings to managing tasks and coordinating communications.

The combination of natural language understanding with structured tool capabilities creates a powerful productivity enhancement that can adapt to various workflows and user needs.
