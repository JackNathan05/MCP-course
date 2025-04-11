
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 2/Lesson 3/_Lesson 3|_Lesson 3]]

---

## Overview

This lesson covers implementing resource providers in MCP, which allow AI models to access data without direct function calls.

## Understanding Resource Contexts

Resources differ from tools in important ways:

- Resources provide data without performing significant computation
- They typically don't have side effects (read-only)
- They are part of the context/request to the AI
- They're similar to GET endpoints in REST APIs

Common resource types include:
- File contents
- Database records
- Configuration data
- User profiles
- System information

The key distinction between tools and resources is that resources are primarily used to provide **context** to the AI model, while tools are used to perform **actions**. Resources are typically provided before the model generates a response, while tools are invoked during or after the model's processing.

## Implementing Resource Providers

Resource providers must:

1. **Define a unique identifier**
   - Descriptive and namespaced ID
   - Clear naming convention
   - Categorization if helpful

2. **Specify the type of data provided**
   - Data format (text, JSON, binary)
   - Structure and schema
   - Size limitations
   - Usage guidelines

3. **Implement data retrieval logic**
   - How to fetch the data
   - Parameter handling
   - Error management
   - Performance considerations

4. **Handle authentication and authorization**
   - Access control checks
   - Permission models
   - Authentication requirements
   - Visibility rules

5. **Manage caching and performance**
   - Caching strategy
   - Freshness vs. performance
   - Resource size limits
   - Pagination if needed

## Resource Provider Example

Here's an example of a file resource provider in Python:

```python
from mcp.server import MCPServer
from mcp.schemas import MCPResource, MCPParameter
import os

# Create server
server = MCPServer()

# Define file resource provider
def file_resource(path: str, max_size_kb: int = 1024):
    """Provides access to file contents"""
    try:
        # Validate path
        if not path or not isinstance(path, str):
            raise ValueError("Path must be a non-empty string")
        
        # Security validation
        abs_path = os.path.abspath(path)
        base_dir = os.path.abspath("/allowed/files/directory")
        
        if not abs_path.startswith(base_dir):
            raise PermissionError(f"Access to files outside {base_dir} is not allowed")
        
        # Check if file exists
        if not os.path.exists(abs_path):
            raise FileNotFoundError(f"File not found: {path}")
        
        # Check if path is a file
        if not os.path.isfile(abs_path):
            raise ValueError(f"Path is not a file: {path}")
        
        # Check file size
        file_size_kb = os.path.getsize(abs_path) / 1024
        if file_size_kb > max_size_kb:
            raise ValueError(f"File exceeds maximum size of {max_size_kb}KB")
        
        # Read file content
        with open(abs_path, 'r') as file:
            content = file.read()
        
        # Determine file type
        file_ext = os.path.splitext(abs_path)[1].lower()
        content_type = get_content_type(file_ext)
        
        # Return file content
        return {
            "content": content,
            "content_type": content_type,
            "file_name": os.path.basename(abs_path),
            "file_size_kb": file_size_kb
        }
        
    except Exception as e:
        # Log error
        print(f"Error accessing file resource: {str(e)}")
        raise

# Helper function to determine content type
def get_content_type(file_ext):
    content_types = {
        ".txt": "text/plain",
        ".md": "text/markdown",
        ".json": "application/json",
        ".csv": "text/csv",
        ".html": "text/html",
        ".xml": "application/xml",
        ".yaml": "application/yaml",
        ".yml": "application/yaml"
    }
    return content_types.get(file_ext, "text/plain")

# Register resource
server.register_resource(
    id="file.content",
    provider=file_resource,
    description="Provides access to file contents",
    parameters=[
        MCPParameter(
            name="path",
            description="Path to the file",
            type="string",
            required=True
        ),
        MCPParameter(
            name="max_size_kb",
            description="Maximum file size in KB",
            type="integer",
            default=1024,
            required=False
        )
    ],
    return_schema={
        "type": "object",
        "properties": {
            "content": {"type": "string"},
            "content_type": {"type": "string"},
            "file_name": {"type": "string"},
            "file_size_kb": {"type": "number"}
        }
    }
)
```

And here's an example of a database record resource provider in TypeScript:

```typescript
import { MCPServer, MCPResource } from 'mcp-server';
import { Database } from './database';

// Create server
const server = new MCPServer();

// Initialize database connection
const db = new Database();

// Register user profile resource
server.registerResource({
  id: 'user.profile',
  name: 'User Profile',
  description: 'Provides user profile information',
  parameters: [
    {
      name: 'user_id',
      description: 'User identifier',
      type: 'string',
      required: true
    },
    {
      name: 'include_preferences',
      description: 'Whether to include user preferences',
      type: 'boolean',
      default: false,
      required: false
    },
    {
      name: 'include_history',
      description: 'Whether to include user history',
      type: 'boolean',
      default: false,
      required: false
    }
  ],
  provider: async (params, context) => {
    try {
      // Get authenticated user from context
      const authenticatedUser = context.user;
      
      // Check if user is requesting their own profile or has admin rights
      if (params.user_id !== authenticatedUser.id && !authenticatedUser.isAdmin) {
        throw new Error('Not authorized to access this user profile');
      }
      
      // Fetch user profile from database
      const userProfile = await db.users.findById(params.user_id);
      
      if (!userProfile) {
        throw new Error(`User not found: ${params.user_id}`);
      }
      
      // Prepare response
      const response = {
        id: userProfile.id,
        name: userProfile.name,
        email: userProfile.email,
        role: userProfile.role,
        created_at: userProfile.createdAt,
        last_login: userProfile.lastLogin
      };
      
      // Include preferences if requested
      if (params.include_preferences) {
        const preferences = await db.preferences.findByUserId(params.user_id);
        response.preferences = preferences;
      }
      
      // Include history if requested
      if (params.include_history) {
        const history = await db.history.findByUserId(params.user_id, { limit: 10 });
        response.history = history;
      }
      
      return response;
    } catch (error) {
      console.error('Error fetching user profile:', error);
      throw error;
    }
  }
});
```

## Resource Caching Strategies

Effective caching improves performance:

1. **Time-based caching (TTL)**
   - Cache resources for a specific time period
   - Set different TTLs based on data volatility
   - Automatically invalidate after expiration

2. **Version-based invalidation**
   - Associate a version with each resource
   - Update version when data changes
   - Compare versions to determine cache validity

3. **Dependency tracking**
   - Track dependencies between resources
   - Invalidate related resources when dependencies change
   - Build dependency graphs for complex relationships

4. **Request-specific caching**
   - Cache based on request parameters
   - Handle parameter variations
   - Optimize for common parameter combinations

5. **Cache size management**
   - Limit cache size to prevent memory issues
   - Implement eviction policies (LRU, LFU)
   - Monitor cache hit rates and adjust strategies

Here's an example of implementing caching for a resource provider:

```python
from mcp.server import MCPServer
from mcp.schemas import MCPResource, MCPParameter
import time

# Simple in-memory cache implementation
cache = {}

# Cache configuration
DEFAULT_TTL = 300  # 5 minutes in seconds

def get_weather_resource(location: str, ttl: int = DEFAULT_TTL):
    """Get weather information for a location with caching"""
    try:
        # Generate cache key based on parameters
        cache_key = f"weather:{location}"
        
        # Check if data is in cache and not expired
        if cache_key in cache:
            cached_data = cache[cache_key]
            
            # Check if cache is still valid
            if time.time() < cached_data['expires_at']:
                print(f"Cache hit for {cache_key}")
                return cached_data['data']
            else:
                print(f"Cache expired for {cache_key}")
        else:
            print(f"Cache miss for {cache_key}")
        
        # If not in cache or expired, fetch fresh data
        weather_data = fetch_weather_data(location)
        
        # Store in cache with expiration time
        cache[cache_key] = {
            'data': weather_data,
            'expires_at': time.time() + ttl
        }
        
        return weather_data
        
    except Exception as e:
        print(f"Error getting weather data: {str(e)}")
        raise

# Function to fetch actual weather data
def fetch_weather_data(location: str):
    # In a real implementation, this would call a weather API
    # This is a simplified example with mock data
    return {
        "location": location,
        "temperature": 22.5,
        "condition": "Partly Cloudy",
        "humidity": 65,
        "wind_speed": 10,
        "updated_at": time.strftime("%Y-%m-%d %H:%M:%S")
    }

# Create server
server = MCPServer()

# Register resource with caching
server.register_resource(
    id="weather.current",
    provider=get_weather_resource,
    description="Provides current weather information for a location",
    parameters=[
        MCPParameter(
            name="location",
            description="City name or geographic coordinates",
            type="string",
            required=True
        ),
        MCPParameter(
            name="ttl",
            description="Cache time-to-live in seconds",
            type="integer",
            default=DEFAULT_TTL,
            required=False
        )
    ]
)
```

## Security Considerations for Resources

Resources often access sensitive data:

1. **Access control checks**
   - User authentication
   - Permission verification
   - Role-based access control
   - Attribute-based access control

2. **Data filtering and sanitization**
   - Filter sensitive fields
   - Sanitize output
   - Prevent information disclosure
   - Apply data masking

3. **Audit logging**
   - Log access attempts
   - Record accessed resources
   - Monitor for suspicious patterns
   - Maintain compliance records

4. **Rate limiting**
   - Prevent abuse
   - Limit request frequency
   - Apply quotas
   - Implement backoff strategies

5. **Content validation**
   - Validate input parameters
   - Check result integrity
   - Ensure data consistency
   - Prevent malformed data

## Best Practices for Resource Providers

1. **Clear documentation**
   - Describe the resource purpose
   - Document parameters
   - Explain access requirements
   - Provide usage examples

2. **Consistent error handling**
   - Use standard error formats
   - Provide helpful error messages
   - Handle expected failure cases
   - Include error details

3. **Performance optimization**
   - Implement appropriate caching
   - Optimize data retrieval
   - Minimize external calls
   - Support pagination for large resources

4. **Security by design**
   - Validate all inputs
   - Apply least privilege principle
   - Protect sensitive data
   - Implement proper authentication

5. **Resource versioning**
   - Support versioned resources
   - Handle backward compatibility
   - Document version differences
   - Provide migration paths

## Summary

Resource providers are essential components of MCP that allow AI models to access data without direct function calls. By implementing effective resource providers with proper caching, security, and error handling, you can enhance the capabilities of AI models while maintaining performance and security.
