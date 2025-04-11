
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 2/Lesson 3/_Lesson 3|_Lesson 3]]

---

## Practical Exercise 2.3

**Objective**: Implement resource providers

### Instructions

1. Extend your MCP server to include resource providers by implementing three different resources:

   a. **File Resource**
   - Create a resource provider that provides access to local text files
   - Include parameters for file path and optional format options
   - Implement security checks to prevent unauthorized access
   - Handle different file types (text, markdown, JSON, etc.)

   b. **Configuration Resource**
   - Implement a resource provider that returns system settings
   - Include parameters to filter which settings to return
   - Support different configuration categories
   - Implement secure access control

   c. **User Profile Resource**
   - Create a resource provider that returns user information
   - Include parameters to specify user and information detail level
   - Implement privacy controls and data filtering
   - Support different views based on the requester's permissions

2. Implement caching for at least one resource:
   - Add time-based caching with configurable TTL
   - Include cache invalidation mechanism
   - Log cache hits and misses
   - Measure and document performance improvements

3. Add appropriate security controls:
   - Validate all input parameters
   - Implement access control checks
   - Filter sensitive information
   - Log access attempts

4. Test the resources with different parameters and permission levels:
   - Valid and invalid parameters
   - Different access levels
   - Cache behavior
   - Error scenarios

### Expected Outcome

A MCP server with three resource providers that:
- Return appropriate data based on parameters
- Implement proper security controls
- Include caching for at least one resource
- Handle errors and edge cases correctly

## Code Example (Java with Spring)

Here's a starter implementation for a file resource provider in Java:

```java
import org.springframework.stereotype.Component;
import io.modelcontextprotocol.server.MCPResource;
import io.modelcontextprotocol.server.MCPResourceProvider;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.TimeUnit;

@Component
public class FileResourceProvider implements MCPResourceProvider {
    
    // Simple cache implementation
    private final ConcurrentHashMap<String, CacheEntry> cache = new ConcurrentHashMap<>();
    
    // Cache entry class
    private static class CacheEntry {
        final Object data;
        final long expirationTime;
        
        CacheEntry(Object data, long ttlMillis) {
            this.data = data;
            this.expirationTime = System.currentTimeMillis() + ttlMillis;
        }
        
        boolean isExpired() {
            return System.currentTimeMillis() > expirationTime;
        }
    }
    
    @Override
    public String getId() {
        return "file_resource";
    }
    
    @Override
    public String getDescription() {
        return "Provides access to local text files";
    }
    
    @Override
    public MCPResource getResource(Map<String, Object> params) {
        // Validate parameters
        if (!params.containsKey("path")) {
            throw new IllegalArgumentException("Missing required parameter: path");
        }
        
        String path = (String) params.get("path");
        long cacheTtl = params.containsKey("cache_ttl") ? 
            ((Number) params.get("cache_ttl")).longValue() : 
            TimeUnit.MINUTES.toMillis(5); // Default 5 minutes
        
        // Check cache first
        String cacheKey = "file:" + path;
        CacheEntry cacheEntry = cache.get(cacheKey);
        
        if (cacheEntry != null && !cacheEntry.isExpired()) {
            return new MCPResource(cacheEntry.data);
        }
        
        // Security validation
        validatePath(path);
        
        try {
            // Read file
            String content = Files.readString(Path.of(path));
            
            // Determine content type
            String contentType = determineContentType(path);
            
            // Prepare response
            Map<String, Object> response = new HashMap<>();
            response.put("content", content);
            response.put("content_type", contentType);
            response.put("file_name", new File(path).getName());
            response.put("size_bytes", Files.size(Path.of(path)));
            
            // Cache result
            cache.put(cacheKey, new CacheEntry(response, cacheTtl));
            
            return new MCPResource(response);
        } catch (IOException e) {
            throw new RuntimeException("Failed to read file: " + e.getMessage());
        }
    }
    
    private void validatePath(String path) {
        // Security validation
        File file = new File(path);
        
        // Check if file exists
        if (!file.exists()) {
            throw new IllegalArgumentException("File does not exist");
        }
        
        // Check if path is a file
        if (!file.isFile()) {
            throw new IllegalArgumentException("Path is not a file");
        }
        
        // Check if file can be read
        if (!file.canRead()) {
            throw new IllegalArgumentException("Cannot read file");
        }
        
        // Restrict to a specific directory for security
        File baseDir = new File("/allowed/files");
        if (!file.getAbsolutePath().startsWith(baseDir.getAbsolutePath())) {
            throw new SecurityException("Access denied to files outside the allowed directory");
        }
    }
    
    private String determineContentType(String path) {
        String lowercase = path.toLowerCase();
        if (lowercase.endsWith(".txt")) return "text/plain";
        if (lowercase.endsWith(".md")) return "text/markdown";
        if (lowercase.endsWith(".json")) return "application/json";
        if (lowercase.endsWith(".html")) return "text/html";
        if (lowercase.endsWith(".xml")) return "application/xml";
        if (lowercase.endsWith(".csv")) return "text/csv";
        if (lowercase.endsWith(".yaml") || lowercase.endsWith(".yml")) return "application/yaml";
        
        // Default to plain text
        return "text/plain";
    }
}
```

## Extension Activities

1. **Advanced Caching**:
   - Implement a more sophisticated caching strategy
   - Add dependency-based cache invalidation
   - Support distributed caching
   - Implement cache statistics and monitoring

2. **Resource Composition**:
   - Create a composite resource that combines data from multiple sources
   - Implement parallel data fetching
   - Add data transformation capabilities
   - Support conditional resource inclusion

3. **Versioned Resources**:
   - Add support for resource versioning
   - Implement compatibility between versions
   - Create migration strategies for clients
   - Document version differences

## Recommended Reading

1. [MCP Resource Implementation Guide](https://modelcontextprotocol.io/docs/guides/resource-implementation/)
   - Resource provider architecture
   - Best practices for data fetching
   - Security considerations

2. [Caching Strategies for MCP](https://modelcontextprotocol.io/docs/guides/caching/)
   - Caching mechanisms
   - TTL selection
   - Invalidation strategies
   - Performance impacts

3. [Security Best Practices for Resources](https://modelcontextprotocol.io/docs/guides/resource-security/)
   - Access control patterns
   - Data filtering
   - Audit logging
   - Rate limiting

## Self-Assessment Questions

Test your understanding by answering these questions:

1. How do resources differ from tools in MCP?
2. What types of data are best suited for resources?
3. What caching strategies can improve resource performance?
4. What security considerations are important for resource providers?
5. How should resource providers handle errors in data retrieval?
6. What parameters should be included for effective resource filtering?
7. How can resources be designed to handle large data sets efficiently?
8. What approaches can be used to control resource access based on user permissions?

## Resource Design Case Study

**Scenario**: You're building an MCP server for a documentation system that needs to provide access to various document types, user information, and system settings.

1. Design three resource providers for this system:
   - Document resource (access to documentation content)
   - User resource (access to user profiles and permissions)
   - Settings resource (access to system configuration)

2. For each resource provider, define:
   - Resource ID and description
   - Parameter schema
   - Return value schema
   - Caching strategy
   - Security controls
   - Error handling approach

3. Create a diagram showing:
   - How these resources interact
   - Data flow between components
   - Security checkpoints
   - Caching mechanisms

## Reflection Questions

Consider and write brief responses to these questions:

1. How would resource providers benefit AI assistants compared to using tools for every data access need?
2. What are the main challenges in implementing efficient resource providers?
3. How would you balance caching for performance against data freshness?
4. What strategies would you use to ensure resource providers remain performant under heavy load?

## Additional Resources

### Resource Implementation Patterns
- [Resource Composition Patterns](https://modelcontextprotocol.io/docs/patterns/resource-composition/)
- [Resource Access Control Patterns](https://modelcontextprotocol.io/docs/patterns/resource-access-control/)

### Performance Optimization
- [Resource Performance Best Practices](https://modelcontextprotocol.io/docs/guides/resource-performance/)
- [Caching Optimization Techniques](https://modelcontextprotocol.io/docs/guides/caching-optimization/)

### Security Guidelines
- [Resource Data Protection](https://modelcontextprotocol.io/docs/guides/data-protection/)
- [Resource Audit Logging](https://modelcontextprotocol.io/docs/guides/audit-logging/)
