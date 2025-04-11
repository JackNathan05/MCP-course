
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 4/Lesson 2/_Lesson 2|_Lesson 2]]

---

## Practical Exercise 4.2

**Objective**: Create an MCP server for enterprise document access

### Instructions

1. Design an MCP server that:
   - Connects to a document repository
   - Implements access control
   - Provides search functionality
   - Logs access for compliance

2. Implement basic functionality with a sample document set:
   - Create a small collection of sample documents (different formats)
   - Implement document indexing and retrieval
   - Add metadata extraction
   - Set up basic search capabilities

3. Test the server with scenarios representing different user roles:
   - Define at least three user roles (e.g., admin, editor, viewer)
   - Create test users for each role
   - Verify that access controls work correctly
   - Test search functionality with various queries

4. Document security and compliance considerations:
   - Identify key security controls
   - Document compliance features
   - Create a data handling policy
   - Outline audit logging capabilities

### Expected Outcome

A working MCP server that:
- Provides secure access to documents
- Implements role-based permissions
- Logs access for compliance purposes
- Demonstrates enterprise security best practices

## Code Example (Role-Based Access Control)

Here's a starter implementation for role-based access control:

```python
# Role-based access control for MCP server
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter
import jwt
import datetime
import logging
import os
from functools import wraps

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# JWT configuration
JWT_SECRET = os.environ.get("JWT_SECRET", "your-secret-key")
JWT_ALGORITHM = "HS256"
JWT_EXPIRATION = 24  # hours

# User database (in-memory for demonstration)
USERS = {
    "admin": {
        "user_id": "user-001",
        "username": "admin",
        "password": "admin-password",  # In production, use hashed passwords
        "name": "Admin User",
        "email": "admin@example.com",
        "roles": ["admin"]
    },
    "editor": {
        "user_id": "user-002",
        "username": "editor",
        "password": "editor-password",
        "name": "Editor User",
        "email": "editor@example.com",
        "roles": ["editor"]
    },
    "viewer": {
        "user_id": "user-003",
        "username": "viewer",
        "password": "viewer-password",
        "name": "Viewer User",
        "email": "viewer@example.com",
        "roles": ["viewer"]
    }
}

# Role permissions
ROLE_PERMISSIONS = {
    "admin": ["search", "view", "create", "update", "delete", "admin"],
    "editor": ["search", "view", "update"],
    "viewer": ["search", "view"]
}

# Create MCP server
server = MCPServer()

# Authentication decorator
def require_auth(*required_permissions):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Get request context
            context = server.get_current_context()
            request = context.request
            
            # Check Authorization header
            auth_header = request.headers.get("Authorization")
            if not auth_header or not auth_header.startswith("Bearer "):
                raise Exception("Authentication required")
            
            token = auth_header[7:]  # Remove "Bearer " prefix
            
            try:
                # Validate JWT token
                payload = jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALGORITHM])
                
                # Check token expiration
                if "exp" not in payload or datetime.datetime.utcnow().timestamp() > payload["exp"]:
                    raise Exception("Token expired")
                
                # Get user roles
                user_roles = payload.get("roles", [])
                
                # Get permissions for user roles
                user_permissions = []
                for role in user_roles:
                    if role in ROLE_PERMISSIONS:
                        user_permissions.extend(ROLE_PERMISSIONS[role])
                
                # Check if user has required permissions
                if required_permissions:
                    has_permission = any(perm in user_permissions for perm in required_permissions)
                    if not has_permission:
                        raise Exception("Insufficient permissions")
                
                # Add user information to context
                context.user = payload
                
                # Log access
                logger.info(f"User {payload.get('username')} accessing {func.__name__}")
                
                # Call the original function
                return await func(*args, **kwargs)
                
            except jwt.DecodeError:
                raise Exception("Invalid token")
            except Exception as e:
                raise Exception(f"Authentication error: {str(e)}")
        
        return wrapper
    return decorator

# Login tool
@server.tool(
    id="auth.login",
    description="Authenticate and get access token",
    parameters=[
        MCPParameter(
            name="username",
            description="Username",
            type="string",
            required=True
        ),
        MCPParameter(
            name="password",
            description="Password",
            type="string",
            required=True
        )
    ]
)
async def login(username, password):
    # Check if user exists
    user = USERS.get(username)
    if not user:
        raise Exception("Invalid username or password")
    
    # In production, use proper password hashing and verification
    if user["password"] != password:
        raise Exception("Invalid username or password")
    
    # Generate JWT token
    expiration = datetime.datetime.utcnow() + datetime.timedelta(hours=JWT_EXPIRATION)
    token_payload = {
        "user_id": user["user_id"],
        "username": user["username"],
        "name": user["name"],
        "email": user["email"],
        "roles": user["roles"],
        "exp": expiration.timestamp()
    }
    
    token = jwt.encode(token_payload, JWT_SECRET, algorithm=JWT_ALGORITHM)
    
    # Log successful login
    logger.info(f"User {username} logged in successfully")
    
    return {
        "token": token,
        "user": {
            "user_id": user["user_id"],
            "username": user["username"],
            "name": user["name"],
            "email": user["email"],
            "roles": user["roles"]
        },
        "expiration": expiration.timestamp()
    }
}

# Document search tool
@server.tool(
    id="documents.search",
    description="Search for documents",
    parameters=[
        MCPParameter(
            name="query",
            description="Search query",
            type="string",
            required=True
        ),
        MCPParameter(
            name="max_results",
            description="Maximum number of results to return",
            type="integer",
            minimum=1,
            maximum=50,
            default=10,
            required=False
        )
    ]
)
@require_auth("search")
async def search_documents(query, max_results=10):
    # In a real implementation, this would search a document database
    context = server.get_current_context()
    user = context.user
    
    # Log document search
    logger.info(f"User {user['username']} searched for: {query}")
    
    # Mock search results
    results = [
        {
            "id": "doc-001",
            "title": "Annual Report 2024",
            "type": "pdf",
            "created": "2024-03-15T10:30:00Z",
            "snippet": "The company achieved record revenue growth of 15% in 2024..."
        },
        {
            "id": "doc-002",
            "title": "Product Roadmap",
            "type": "docx",
            "created": "2024-02-20T14:45:00Z",
            "snippet": "Key features planned for Q3 include AI integration and..."
        },
        {
            "id": "doc-003",
            "title": "HR Policies",
            "type": "pdf",
            "created": "2023-11-05T09:15:00Z",
            "snippet": "Updated remote work policies effective January 1, 2024..."
        }
    ]
    
    # Limit results
    results = results[:max_results]
    
    return {
        "query": query,
        "total_results": len(results),
        "results": results
    }

# Document view tool
@server.tool(
    id="documents.view",
    description="View a document by ID",
    parameters=[
        MCPParameter(
            name="document_id",
            description="Document ID",
            type="string",
            required=True
        )
    ]
)
@require_auth("view")
async def view_document(document_id):
    # In a real implementation, this would retrieve a document from a database
    context = server.get_current_context()
    user = context.user
    
    # Log document view
    logger.info(f"User {user['username']} viewed document: {document_id}")
    
    # Mock document data
    documents = {
        "doc-001": {
            "id": "doc-001",
            "title": "Annual Report 2024",
            "type": "pdf",
            "created": "2024-03-15T10:30:00Z",
            "content": "This is a sample annual report with financial data...",
            "metadata": {
                "author": "Finance Department",
                "classification": "Internal",
                "pages": 45
            }
        },
        "doc-002": {
            "id": "doc-002",
            "title": "Product Roadmap",
            "type": "docx",
            "created": "2024-02-20T14:45:00Z",
            "content": "This document outlines the product development plan...",
            "metadata": {
                "author": "Product Team",
                "classification": "Confidential",
                "pages": 12
            }
        },
        "doc-003": {
            "id": "doc-003",
            "title": "HR Policies",
            "type": "pdf",
            "created": "2023-11-05T09:15:00Z",
            "content": "This document contains the company's HR policies...",
            "metadata": {
                "author": "HR Department",
                "classification": "Internal",
                "pages": 28
            }
        }
    }
    
    # Check if document exists
    if document_id not in documents:
        raise Exception(f"Document not found: {document_id}")
    
    # Get document
    document = documents[document_id]
    
    # Check document classification and user roles
    # In a real implementation, this would be more sophisticated
    if document["metadata"]["classification"] == "Confidential" and "admin" not in user["roles"] and "editor" not in user["roles"]:
        raise Exception("Access denied: Insufficient permissions for this document")
    
    return document

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"Enterprise MCP server running on port {port}")
```

## Extension Activities

1. **Document Indexing Enhancement**:
   - Implement full-text indexing for documents
   - Add support for multiple document formats (PDF, Office, etc.)
   - Implement metadata extraction
   - Create a document classification system

2. **Advanced Access Control**:
   - Implement attribute-based access control (ABAC)
   - Add document-level permissions
   - Create dynamic permission evaluation
   - Implement approval workflows for sensitive documents

3. **Compliance Reporting**:
   - Create comprehensive audit logging
   - Implement compliance reports
   - Add data access monitoring
   - Create user activity dashboards

## Recommended Reading

1. [MCP Enterprise Integration Guide](https://modelcontextprotocol.io/docs/guides/enterprise-integration/)
   - Best practices for enterprise deployments
   - Security patterns for MCP servers
   - Compliance considerations
   - Performance optimization

2. [Document Repository Integration](https://modelcontextprotocol.io/docs/guides/document-integration/)
   - Document processing techniques
   - Metadata extraction strategies
   - Search optimization
   - Content handling

3. [Security Best Practices for MCP](https://modelcontextprotocol.io/docs/guides/security/)
   - Authentication and authorization
   - Data protection
   - Secure communication
   - Threat modeling

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What are the main challenges when connecting AI assistants to enterprise data?
2. How should document access be managed in MCP servers?
3. What security controls are essential for enterprise MCP implementations?
4. What compliance considerations must be addressed?
5. How should data access logging be implemented?
6. What are the different approaches to implementing access control?
7. How can MCP servers facilitate compliance with data protection regulations?
8. What strategies can be used to ensure secure document retrieval?

## Case Study: Financial Services Document System

**Scenario**: A financial services company needs to implement an AI assistant that can access internal documents securely while maintaining regulatory compliance.

### Requirements:
- Document repository integration with access controls
- Regulatory compliance (SOX, GDPR, SEC regulations)
- Multi-level security classification
- Comprehensive audit logging
- Secure client-side data access

### Tasks:
1. **Design the system architecture**:
   - Draw a diagram showing components and data flow
   - Identify security boundaries
   - Design authentication and authorization
   - Plan for compliance requirements

2. **Define access control model**:
   - Identify user roles and permissions
   - Define document classification levels
   - Create access control policies
   - Design approval workflows for sensitive data

3. **Plan compliance features**:
   - Identify required audit logs
   - Design compliance reports
   - Plan data retention policies
   - Create security incident response processes

4. **Prototype a key component**:
   - Implement a core component (e.g., document access with RBAC)
   - Test with realistic scenarios
   - Document security controls
   - Evaluate compliance effectiveness

## Compliance Documentation Template

Create compliance documentation for your MCP server implementation:

### 1. Security Controls

| Control ID | Description | Implementation | Compliance Mapping |
|------------|-------------|----------------|-------------------|
| SEC-01 | Authentication | JWT-based authentication with MFA | NIST 800-53 IA-2 |
| SEC-02 | Authorization | Role-based access control for all endpoints | NIST 800-53 AC-3 |
| SEC-03 | Audit Logging | Comprehensive logging of all data access | NIST 800-53 AU-2 |
| SEC-04 | Data Encryption | TLS 1.3 for all communications | NIST 800-53 SC-8 |
| SEC-05 | Session Management | Secure token handling with expiration | NIST 800-53 AC-12 |

### 2. Compliance Matrix

| Requirement | Control ID | Implementation Details | Validation Method |
|-------------|------------|------------------------|-------------------|
| Access Control | SEC-02 | RBAC model with document-level permissions | Automated testing of permission scenarios |
| Audit Logging | SEC-03 | JSON-structured logs with all relevant details | Manual review of log completeness |
| Data Protection | SEC-04 | Encryption in transit and at rest | Automated scanning and penetration testing |
| Authentication | SEC-01 | Multi-factor authentication with strong password policy | Authentication flow testing |
| Session Security | SEC-05 | Short-lived tokens with secure storage | Security review of token handling |

### 3. Data Handling Policy

1. **Data Classification**:
   - Public: Freely accessible information
   - Internal: Company-wide information
   - Confidential: Department-specific information
   - Restricted: Need-to-know basis only

2. **Access Requirements**:
   - Public: No authentication required
   - Internal: Valid user account required
   - Confidential: Department membership required
   - Restricted: Explicit approval required

3. **Handling Procedures**:
   - Data minimization in all responses
   - Automatic redaction of sensitive information
   - Explicit logging of all access to confidential data
   - Approval workflow for restricted data

## Additional Resources

### Enterprise Integration
- [MCP Enterprise Reference Architecture](https://modelcontextprotocol.io/docs/reference/enterprise-architecture/)
- [Document Processing Libraries](https://modelcontextprotocol.io/docs/resources/document-processing/)
- [Security Testing Tools](https://modelcontextprotocol.io/docs/resources/security-testing/)

### Compliance Resources
- [Compliance Templates](https://modelcontextprotocol.io/docs/templates/compliance/)
- [Audit Log Schemas](https://modelcontextprotocol.io/docs/schemas/audit-logs/)
- [Security Control Frameworks](https://modelcontextprotocol.io/docs/frameworks/security-controls/)

### Implementation Examples
- [RBAC Implementation Patterns](https://modelcontextprotocol.io/docs/patterns/rbac/)
- [Document Repository Connectors](https://modelcontextprotocol.io/docs/connectors/document-repositories/)
- [Compliance Reporting Tools](https://modelcontextprotocol.io/docs/tools/compliance-reporting/)
